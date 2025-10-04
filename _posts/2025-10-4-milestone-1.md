---
layout: post
title: IFT6758 Demo Post
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

## Question 4 - Simple Visualizations

## Question 5 - Advanced Visualizations