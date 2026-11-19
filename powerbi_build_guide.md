# Power BI build guide: Spotify Track Popularity

Follow these steps in Power BI Desktop. Use `data/spotify_tracks_cleaned.csv` (28,352 rows, one per unique track). The checks at the end tell you whether each number matches the data.

## 1. Load the data
1. Home > Get data > Text/CSV > choose `spotify_tracks_cleaned.csv` > Transform Data.
2. In Power Query set types: `track_popularity`, `release_year` = Whole number; `danceability`, `energy`, `valence`, `acousticness`, `duration_min` = Decimal number; `release_date` = Date.
3. Rename the table to **Tracks**. Close & Apply.

The cleaned file already has `release_year`, `duration_min`, `popularity_tier` and `mood`. If you prefer to build them in Power BI, use the formulas in section 3.

## 2. Measures (Modeling > New measure)
```DAX
Total Tracks = DISTINCTCOUNT(Tracks[track_id])
Total Artists = DISTINCTCOUNT(Tracks[track_artist])
Avg Popularity = AVERAGE(Tracks[track_popularity])
Avg Popularity (All Genres) = CALCULATE([Avg Popularity], ALL(Tracks[playlist_genre]))
Zero Popularity Tracks = CALCULATE(COUNTROWS(Tracks), Tracks[track_popularity] = 0)
Zero Popularity % = DIVIDE([Zero Popularity Tracks], [Total Tracks])
High Popularity % = DIVIDE(CALCULATE(COUNTROWS(Tracks), Tracks[track_popularity] > 60), [Total Tracks])
Avg Track Length (min) = AVERAGE(Tracks[duration_min])
```

## 3. Calculated columns (only if you do not use the pre-built ones)
```DAX
Popularity Tier =
SWITCH(TRUE(),
  Tracks[track_popularity] = 0, "0 Not streamed",
  Tracks[track_popularity] <= 30, "1 Low (1-30)",
  Tracks[track_popularity] <= 60, "2 Medium (31-60)",
  "3 High (61-100)")

Mood =
SWITCH(TRUE(),
  Tracks[valence] >= 0.5 && Tracks[energy] >= 0.5, "Happy / energetic",
  Tracks[valence] <  0.5 && Tracks[energy] >= 0.5, "Angry / intense",
  Tracks[valence] >= 0.5 && Tracks[energy] <  0.5, "Calm / positive",
  "Sad / low energy")
```

## 4. Page 1: Overview
| Visual | Fields |
| :--- | :--- |
| 5 Cards | Total Tracks, Total Artists, Avg Popularity, Zero Popularity %, Avg Track Length (min) |
| Clustered bar | Axis: playlist_genre, Values: Avg Popularity (sort descending); add a constant line at Avg Popularity (All Genres) |
| Clustered bar | Axis: playlist_genre, Values: Total Tracks |
| Column chart | Axis: popularity_tier, Values: Total Tracks |
| Line chart | Axis: release_year (filter 2000 to 2020), Values: Total Tracks |
| Bar chart | Axis: track_artist, Values: Avg Popularity; Top N filter 8 by Avg Popularity; visual filter Total Tracks >= 10 |
| Slicers | playlist_genre, release_year, popularity_tier |

## 5. Page 2: Audio features
| Visual | Fields |
| :--- | :--- |
| Matrix (heat map) | Rows: playlist_genre; Values: average of danceability, energy, valence, acousticness; Format > Cell elements > Background colour (gradient) |
| Bar chart | Axis: playlist_genre, Values: Zero Popularity %; conditional colour above 9.2% |
| Line chart | Axis: release_year (2010 to 2020), Values: Avg Popularity |
| Bar chart | Axis: mood, Values: Avg Popularity, tooltip: Total Tracks |
| Scatter | X: duration_min, Y: track_popularity (average), Details: playlist_genre |

Power BI has no built-in correlation visual. Use the correlation values in the README (calculated in Python) as a text box, or add a Python visual with `df.corr()`.

## 6. Formatting
Background `#fcfcfb`, text `#0b0b0b`, blue `#2a78d6` for normal bars, orange `#eb6834` for values above the overall average. Titles are left aligned. Turn off gridlines on the bar charts. Page size 16:9.

## 7. Check your numbers
| Check | Expected |
| :--- | :--- |
| Total Tracks | 28,352 |
| Total Artists | 10,692 |
| Avg Popularity | 39.3 |
| Zero Popularity % | 9.2% |
| Avg Track Length | 3.78 min |
| Avg Popularity: POP / RAP / LATIN / ROCK / R&B / EDM | 45.9 / 41.8 / 41.4 / 39.7 / 35.9 / 30.7 |
| Tracks by tier (0 / low / medium / high) | 2,616 / 6,819 / 12,943 / 5,974 |
