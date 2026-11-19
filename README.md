# Spotify Track Popularity Dashboard (Power BI)

A two-page Power BI dashboard design for a public Spotify tracks dataset: what separates popular tracks from the rest, by genre, release year, artist and audio features. All figures below were calculated from the data in Python and written so you can reproduce them as DAX measures in Power BI Desktop (`powerbi_build_guide.md`). The dashboard images are Python previews of the layout, not Power BI exports.

**Data:** `spotify_songs.csv`, the TidyTuesday release of 2020-01-21 (from the `rfordatascience/tidytuesday` GitHub repository): 32,833 rows, 23 columns, one row per track per playlist. It is a snapshot from early 2020, and `track_popularity` is Spotify's 0 to 100 score at that time.

---

## Executive Summary

- **Scale:** 28,352 unique tracks by 10,692 artists, average popularity **39.3** out of 100 (median 42).
- **Genre:** Pop is the most popular genre (**45.9**), then Rap (41.8), Latin (41.4), Rock (39.7) and R&B (35.9). **EDM is lowest at 30.7.**
- **Top tier:** 33.5% of Pop tracks score above 60, against 6.5% of EDM tracks. Overall, 21.1% of tracks are in the top tier.
- **Not streamed:** 2,616 tracks (**9.2%**) have a popularity of 0. EDM has the most (13.3%), then R&B (10.8%); Pop has the fewest (6.1%).
- **Audio features explain little:** every correlation with popularity is between -0.14 and +0.09. Longer tracks (-0.14), more instrumental tracks (-0.12) and higher-energy tracks (-0.10) score a little lower; acoustic tracks (+0.09) a little higher.
- **Mood:** "Sad / low energy" tracks average 43.0 and "Angry / intense" tracks 38.0 (valence and energy split at 0.5).
- **Release year:** 2019 is the biggest year in the data (7,406 tracks, 26.1%), with an average popularity of 47.4. Recent releases score higher, so year matters more than sound.
- **Artists:** among artists with 10+ tracks, Billie Eilish leads (77.0 across 26 tracks), then Tones and I (74.6), Green Day (73.2), Camila Cabello (72.2) and Khalid (71.6).

---

## Page 1: Overview

| Element | Result |
| :--- | :--- |
| Unique tracks | 28,352 |
| Artists | 10,692 |
| Average popularity | 39.3 |
| Zero-popularity tracks | 2,616 (9.2%) |
| Average track length | 3.78 minutes |
| Tracks by genre | Rap 5,398; Pop 5,132; EDM 4,877; R&B 4,504; Rock 4,305; Latin 4,136 |
| Tracks by tier | Not streamed 2,616; Low (1-30) 6,819; Medium (31-60) 12,943; High (61-100) 5,974 |
| Slicers | Genre, Release Year, Popularity Tier |

## Page 2: Audio features

| Feature | Correlation with popularity |
| :--- | :--- |
| Track length | -0.14 |
| Instrumentalness | -0.12 |
| Energy | -0.10 |
| Liveness | -0.05 |
| Tempo, Speechiness | 0.00, +0.01 |
| Valence | +0.02 |
| Loudness | +0.04 |
| Danceability | +0.05 |
| Acousticness | +0.09 |

Average audio profile (danceability, energy, valence, acousticness): Pop 0.64 / 0.70 / 0.50 / 0.17; Rap 0.72 / 0.65 / 0.51 / 0.20; Latin 0.71 / 0.71 / 0.61 / 0.21; Rock 0.52 / 0.73 / 0.53 / 0.15; R&B 0.67 / 0.59 / 0.54 / 0.26; EDM 0.66 / 0.81 / 0.40 / 0.08.

---

## Data cleaning (what changed and why)

| Step | Rows | Reason |
| :--- | :--- | :--- |
| Raw file | 32,833 | |
| Dropped rows with missing track name, artist or album (5 each, same rows) | 32,828 | Cannot identify the track |
| Kept one row per `track_id` | **28,352** | The same track appears in several playlists (4,476 repeats); counting it more than once would inflate every total |

Other changes: `release_year` parsed from `track_album_release_date` (formats of 4, 7 and 10 characters, no failures); `duration_min` = `duration_ms` / 60,000; `popularity_tier` and `mood` added; genre names upper-cased. **Decision to know about:** when a track sits in playlists of different genres, the row with the highest popularity is kept, so the genre of that row is used. Genre figures therefore describe each track once, not every playlist it appears in.

---

## Recommendations

- **For playlist and A&R decisions, weight genre and release timing above audio features.** The sound features explain very little, so "make it more danceable" is not supported by this data.
- **Pop, Rap and Latin carry the highest share of top-tier tracks.** Treat EDM with care: only 6.5% reach the top tier and 13.3% are not streamed at all.
- **Check the 2,616 zero-popularity tracks** before promoting them; some are probably new releases and some may be removed or region-locked (the data cannot say which).
- **Use artist averages with a minimum track count** (10+ here) so one big hit does not decide the ranking.

## Limitations

- Popularity is a snapshot score from early 2020 and favours recent releases, so older tracks are not directly comparable.
- Correlations are small and describe association only; they do not show what makes a track popular.
- Genre comes from playlist labels, not from the artist or track itself.
- The data has no streams, revenue or listener data, so nothing here measures money.
- Previews are drawn in Python; build the live report in Power BI Desktop with the guide.

---

## Repository structure

```text
PowerBI_spotify_track_analytics/
├── data/
│   ├── spotify_songs.csv              # raw, 32,833 rows
│   └── spotify_tracks_cleaned.csv     # 28,352 unique tracks
├── analysis/
│   ├── 01_clean_spotify.py            # cleaning script
│   └── 02_dashboard_preview.py        # draws the preview images
├── screenshots/
│   ├── spotify_dashboard_page1_preview.png
│   └── spotify_dashboard_page2_preview.png
├── presentation/Spotify Track Popularity Analytics.pptx
├── powerbi_build_guide.md             # steps, DAX measures, numbers to check
└── README.md
```
