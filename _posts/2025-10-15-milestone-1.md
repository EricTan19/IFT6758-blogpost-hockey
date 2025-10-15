---
layout: post
title: IFT6758 Demo Post
use_math: true
---

## Question 1 - Data Acquisition

To retrieve play-by-play data, simply use the function retrieve_target_year_schedule(target_year). This function will call retrieve_playoffs_games(target_year) and retrieve_regular_season_games(target_year), since each game type has different logic to retrieve the data.

It is important to note that different game types have different logic because the gameId is constructed differently!

### Playoff games

To retrieve the playoffs games data, the function will make a GET api request call to "https://api-web.nhle.com/v1/playoff-series/carousel/{season_id}" to retrieve all of the series letters (numberOfRounds) for a single season. It will then make another GET api request call to "https://api-web.nhle.com/v1/schedule/playoff-series/{season_id}/{i}" which will retrieve all the gameIds/nbOfGames for a certain round (seriesLetter). The play-by-play data will then be retrieved by looping across all the gameIds using retrieve_play_by_play_data(filepath).

### Regular season games

Similarly, to retrieve regular season data, the function will make a GET api request call to "https://api.nhle.com/stats/rest/en/season", which will retrieve all of the seasonIds. We then obtain the number of 'totalRegularSeasonGames' and loop through all of the gameIds using retrieve_play_by_play(filepath) from 1 to #totalRegularSeasonGames.

The retrieve_play_by_play(filepath) function creates folders and json files if they do not exist. It makes a GET api request call to "https://api-web.nhle.com/v1/gamecenter/{game_id}/play-by-play" to retrieve the play-by-play data for a specific gameId obtained from the argument. The play-by-play data is stored in its respective filepath as a .json file.

## Question 2 - Interactive Debugging Tool

![Screenshot](milestone1_q2.png)

This interactive debug tool lets you retrieve downloaded games, filter by year and game type, and inspect specific events. If event coordinates are available, they are shown on the NHL rink along with game score and other details.

### Code

```python
import ipywidgets as widgets
from IPython.display import display, clear_output
import datetime
import json
from pathlib import Path
from pprint import pprint
import matplotlib.pyplot as plt
import matplotlib.image as mpimg


RINK_IMG = Path("nhl_rink.png")
RINK_LEN = 200.0
RINK_WIDTH = 85.0

current_year = datetime.datetime.now().year

year_selector = widgets.Dropdown(
    options=list(range(1900, current_year + 2)),
    value=current_year,
    description="Year:",
)

season = widgets.Dropdown(options=list(range(1900, current_year + 2)), value=current_year, description="Year: ")
game_type = widgets.Dropdown(options=["playoffs", "regseason"], value="playoffs", description="Game Type: ")
game_id = widgets.Dropdown(options=[], description="Game: ")
events = widgets.IntSlider(value=1, min=1, max=1, step=1, description="Event: ")

score_out = widgets.Output()
rink_out = widgets.Output()
out = widgets.Output()

with rink_out:
    fig, ax = plt.subplots(figsize=(12, 6))
    img = mpimg.imread(RINK_IMG)
    ax.imshow(img, extent=[-RINK_LEN/2, RINK_LEN/2, -RINK_WIDTH/2, RINK_WIDTH/2], zorder=0)
    ax.set_xlim(-RINK_LEN/2, RINK_LEN/2)
    ax.set_ylim(-RINK_WIDTH/2, RINK_WIDTH/2)
    ax.set_aspect("equal")
    ax.set_xlabel("feet"); ax.set_ylabel("feet")

    marker = ax.scatter([], [], s=120, zorder=2, c="green")
    marker.set_visible(False)
    plt.show()

def retrieve_game_ids(season, game_type):
    """
    Retrieve all the game ids for a specific season and game_type

    Args:
        season: season of the game
        game_type: playoffs or regseason

    Returns:
        all_game_ids for a specific season and game_type in database (local)
    """

    path = Path(f"../data/{season}/{game_type}/")
    all_game_ids = [p.name for p in path.rglob("*") if p.is_dir()]

    return all_game_ids

# create new path to retrieve data
def load_play_by_play_data(season, game_type, game_id):
    """
    Retrieve the play-by-play data from a game

    Args:
        season: season of the game
        game_type: playoffs or regseason
        game_id: gameId for the specific game

    Returns:
        None
    """

    path = Path(f"../data/{season}/{game_type}/{game_id}/{game_id}_play_by_play.json")
    if not path.exists():
        return {"error": f"File not found: {path}"}
    with path.open("r", encoding="utf-8") as f:
        return json.load(f)


def retrieve_events(data):
    """
    Retrieve events from the data dump provided by the api call

    Args:
        data: data dump from api call

    Returns:
        all the events from the game
    """


    events_list = [play for play in data.get("plays", [])]

    nbOfPlays = max(1, len(events_list))
    events.max = nbOfPlays
    if events.value < 1 or events.value > nbOfPlays:
        events.value = 1
    return events_list


def refresh(_=None):
    """
    Function that refreshes on any variable changes

    Args:
        optional argument to be used as a callback or direct call

    Returns:
        None

    """
    with out:
        clear_output(wait=True)
        data = load_play_by_play_data(season.value, game_type.value, game_id.value)
        events_list = retrieve_events(data)

        play = events_list[events.value - 1]

        with score_out:
            clear_output(wait=True)
            details   = play.get("details", {}) or {}

            homeTeam = (data.get("homeTeam") or {}).get("abbrev", "HOME")
            awayTeam = (data.get("awayTeam") or {}).get("abbrev", "AWAY")

            homeScore, awayScore = last_known_score(events_list, events.value - 1)

            timeInPeriod  = play.get("timeInPeriod", "")
            period  = (play.get("periodDescriptor") or {}).get("number", "")

            if homeScore is not None and awayScore is not None:

                print(f"Score @ {timeInPeriod} P{period}: {homeTeam} {homeScore} – {awayTeam} {awayScore}")
            else:
                print(f"Score @ {timeInPeriod} P{period}: (no score on this play)")

        pprint(play)

        details = play.get("details", {})

        if "xCoord" in details and "yCoord" in details:
            x, y = details["xCoord"], details["yCoord"]
            plot_rink_with_point(x, y)
        else:
            plot_rink_with_point(None, None)

def last_known_score(events_list, idx):
    """
    Retrieve last known score for a certain play

    Args:
        events_list: list of events for a specific game

        idx: index of the play
    """

    for j in range(idx, -1, -1):
        detail = (events_list[j].get("details") or {})
        homeScore = detail.get("homeScore")
        awayScore = detail.get("awayScore")
        if homeScore is not None and awayScore is not None:
            return homeScore, awayScore
    return 0, 0

def plot_rink_with_point(x, y):
    """
    Plot marker on the rink image

    Args:
        x and y coordinates of the play

    Returns:
        None
    """
    if x is None or y is None:
        marker.set_visible(False)
    else:
        marker.set_offsets([[x, y]])
        marker.set_visible(True)

    fig.canvas.draw()

    with rink_out:
        from IPython.display import clear_output, display
        clear_output(wait=True)
        display(fig)



def repopulate_game_ids(_=None):
    """
    Refresh the Game ID choices based on the current season and game type

    Args:
        optional argument to be used as a callback or direct call

    Returns:
        None
    """
    game_ids = retrieve_game_ids(season.value, game_type.value)
    game_id.options = game_ids
    if game_ids:
        game_id.value = game_ids[0]
    refresh()


for w in (season, game_type, game_id):
    w.observe(refresh, names="value")

season.observe(repopulate_game_ids, names="value")
game_type.observe(repopulate_game_ids, names="value")
game_id.observe(refresh, names="value")
events.observe(refresh, names="value")

controls = widgets.HBox([season, game_type, game_id, events])
ui = widgets.VBox([controls, score_out, rink_out, out])   # controls on top, then rink, then output

display(ui)
```

## Question 3 - Tidy Data

After retrieving and exploring the data, the next step is to clean and format the play-by-play information into a well-structured pandas DataFrame. This DataFrame will contains all the relevant details for events of type “shot” and “goal”.

### 1. DataFrame

![Screenshot](milestone1_q3.png)

It is often useful to create additional features from the data available in the dataset; this process is known as feature engineering. These new features can then be used to generate insightful visualization or train models.

### 2. Feature engineering

First, we could use the shot coordinates (x, y) to calculate metrics such as the **distance** and **angle** of the shot relative to the goal. The **distance** can be calculated using the euclidean distance from the goal's coordinate which is typically (89, 0). Similarly, the **angle** of the shot can be calculated using basic trigonometry.

Second, we can add a **shot_context** column to capture the sequence leading up to each attempt. This feature would take one of three values—**rebound**, **rush**, or **normal**—based on the previous event. Classify a shot as a **rebound** if the prior event was a shot or goal by the same team within ≤ 3 seconds. Label it **rush** if the prior event was by the other team (change of possession) and the shot occurs within ≤ 7 seconds. If neither condition holds, tag it as **normal**.

Third, we could add a **shooter_shots** column which keep tracks of the total number of shots the player has taken since the beginning of the game and a **goalie_faced_shots** column which counts the total number of shots faced so far by the goalie. These features capture potential hot-hand effects for shooters and fatigue effects for goalies, which can influence shot outcomes.

## Question 4 - Simple Visualizations

### 1. Comparing the shot types over all teams

![Screenshot](milestone1_q4_1.png)

From the figure above, we see that the most dangerous type of shots (highest goal probability) are the **bat** (28%) and the **poke** (22%). These shots have low counts which can contribute to inflate the rates, but these are the most dangerous based on the data we have.
Among the common shot types, **deflected** and **tip-in** lead the way with a goal probability around 20%, while **backhand** and **snap** sit next at around 15%.

Also, the histogram shows that **wrist** shots are by far the most common, followed by **snap** shots and **slap** shots. Next, we have the **backhand** and **tip-in** which are also fairly common.

### 2. Goal percentage as a function of distance

![Screenshot](milestone1_q4_2.png)

The figure illustrates the correlation between the **goal probability** and the **distance** from which the shot was taken across different seasons. Using shared axes for all seasons makes comparisons straightforward and the binned plot makes the relationship immediately interpretable but can suffers from small samples. As we the **distance** increases, the **goal probability** drops rapidly. Shots that are within 5 ft has very high rates but this can also be attributed to small samples. From 10ft to 60ft, the goal probability decreases linearly from 20% to less than 5%. Beyond this point, the bumps are mostly noise due to the small number of samples.
Also, the curves seems to remains almost identical through the different seasons. Thus, we can conclude that there's a clear correlation between these two variables which makes sense.

### 3. Goal percentage as a function of both distance and shot type

![Screenshot](milestone1_q4_3.png)

On the above figure, we plot the **goal probability** as a function of both the **distance** and the **shot type** for the 2024 season. This view illustrate the previous relationship but individually for every type of shots. Since some type of shots are really rare, this graph suffers from small samples. However, we can identify some interesting tendency. Some shots have higher probability when the **distance** is really small while other still performs well in moderate **distance** like the slap shot for example.

## Question 5 - Advanced Visualizations

### 1. Offensive zone plots 

{% include shotmap_dropdown.html %}

### 2. Plot Interpretation 

The shot maps show that most teams generate offense from the slot area in front of the net, where chances are most dangerous. You can also see differences in team strategies some spread shots around the zone while others focus heavily near the crease. When efficiency is added, it becomes clear which teams make good use of their chances and which simply shoot a lot without scoring as much. Comparing seasons also highlights how a team’s approach can shift over time.

### 3. Colorado Avalanche Analysis

When you look at Colorado’s shot map from the 2016-2017 season, it’s immediately clear the team was struggling. Most of their shots came from the outside or the middle of the slot instead of right in front of the net. That usually means they couldn’t get inside or create many dangerous chances. You can also see they were shooting mostly from the right side of the zone, which made their offense pretty predictable  it doesn’t feel like a team that could attack in different ways or really apply sustained pressure.

There isn’t much of a shot cluster near the crease, which shows they weren’t generating rebounds or screens, and probably weren’t winning many battles in front of the net. A lot of their shots came from “safe” areas with low scoring probability, like the blue line or along the boards. When you remember they finished last in the league that year in goals, shooting percentage, and almost everything else the shot map matches that reality: it was a team that couldn’t get to dangerous areas and lacked offensive variety.

Now if you compare that to their 2020-2021 shot map, the difference is pretty striking. There’s a lot more activity right in front of the net and in the slot, and the shots are more evenly spread across the zone. It looks like a team that can actually get inside with the puck and sustain pressure instead of just shooting from the outside. You also don’t see that same one-sided pattern  the attack feels more complete and harder to defend.

And when you line it up with the standings, it makes perfect sense. In 2016-2017 they were the worst team in the league. By 2020-2021, they were one of the best. The shot maps reflect that shift from a team taking whatever outside shots they could get to one that controls the zone and lives in dangerous areas.


### 4. Buffalo Sabres vs Tampa Bay Lightning

In all seasons, the Buffalo Sabres shot map shows limited red zones, indicating they generated fewer high-quality scoring chances than the league average, often shooting from the perimeter rather than from dangerous areas near the net. In contrast, the Tampa Bay Lightning’s maps displays strong red concentrations in the slot and crease areas, reflecting a strategy centered on creating high-danger chances close to goal.

By 2021, Tampa Bay’s map shows an even tighter cluster of red directly in front of the net, highlighting their focus on efficiency, taking fewer but better-quality shots. This offensive precision likely contributed to their back-to-back Stanley Cup wins.

Overall, the Lightning’s success appears tied to generating quality chances from prime scoring zones, while Buffalo's struggles stem from less efficient shot placement. However, these maps provide only part of the story, as factors like defense, goaltending, and player skill also play key roles in overall performance.
