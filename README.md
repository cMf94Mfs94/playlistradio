# Playlist Radio

A browser-based tool that generates AI-powered "radio" playlists for Plex based on your existing playlists. Uses OpenAI and/or Last.FM APIs to discover similar tracks from your music library.

## Overview

Playlist Radio analyzes tracks in your existing Plex playlists and uses AI to recommend similar songs you already own. It generates a Python script that creates a new playlist in Plex with all the discovered tracks.

## Features

- **Dual API Support**: Use OpenAI, Last.FM, or both simultaneously for recommendations
- **Smart Track Selection**: Option to use a random percentage of tracks from large playlists
- **Customizable Prompts**: Full control over OpenAI prompts with variable substitution
- **Duplicate Detection**: Automatically removes duplicate recommendations across sources
- **Freshness Control**: Skip tracks played within a configurable time period
- **API Call Preview**: See estimated API usage before generating
- **Overwrite Protection**: Warns if playlist name already exists
- **Browser-Based**: No server setup required - runs entirely in your browser

## Setup Instructions

### Prerequisites

1. **Plex Media Server** with music library
2. **Plex API Token** - [How to find your token](https://support.plex.tv/articles/204059436-finding-an-authentication-token-x-plex-token/)
3. **OpenAI API Key** (optional but recommended) - [Get one here](https://platform.openai.com/api-keys)
4. **Last.FM API Key** (optional) - [Get one here](https://www.last.fm/api/account/create)
5. **Python 3** with PlexAPI library: `pip install PlexAPI`

### Getting Started

1. Open `index.html` in any modern web browser
2. Click the **Settings** button in the top-right corner
3. Configure the following settings:

#### Required Settings

- **Plex Server URL**: Your Plex server address (e.g., `http://192.168.1.100:32400`)
- **Plex API Token**: Your authentication token
- **Music Section ID**: The library section number for your music (check Plex web URL when viewing music library)

#### Recommended Settings

- **Default similar tracks per track**: How many similar tracks to request per playlist track (default: 10)
- **Not played in last X days**: Skip tracks played recently (default: 30)
- **OpenAI API Key**: Your OpenAI API key (enables AI recommendations)
- **ChatGPT Model**: Which model to use (default: `gpt-3.5-turbo`, also supports `gpt-4`, `gpt-4-turbo`, etc.)

#### Optional Settings

- **Last.FM API Key**: Enable Last.FM as an additional recommendation source

4. Click **Save** to store settings in your browser's localStorage

## How to Use

### Basic Workflow

1. **Select a Playlist**: Click "View" next to any playlist from the main list
2. **Configure Options**: Adjust settings for this specific generation
3. **Generate**: Click "Generate New Playlist" button
4. **Download Script**: The browser will automatically download `createplaylist.py`
5. **Run Script**: Execute `python createplaylist.py` to create the playlist in Plex

### Generation Options

#### New Playlist Name
- Default: `[Original Name] Radio`
- **Overwrite Protection**: If name already exists, you'll be prompted to confirm

#### Similar Tracks Per Track
- Number of recommendations to request per playlist track
- Can be overridden from the default setting per generation

#### APIs (Select at Least One)
- **OpenAI**: Uses ChatGPT to generate contextual recommendations
- **Last.FM**: Uses Last.FM's track similarity database
- **Both**: Combines results from both sources with deduplication

#### Not Played in Last X Days
- Filters out recently played tracks from recommendations
- Can be overridden from the default setting per generation

#### OpenAI Prompt
Customize how the AI generates recommendations using these variables:
- `{TRACK}`: Replaced with the track title
- `{ARTIST}`: Replaced with the artist name
- `{N}`: Replaced with the similar tracks per track value

**Default prompt:**
```
I like the following song: {TRACK} by {ARTIST}
Name {N} tracks by different artists that I would also like. Provide each as 'Artist - Track Title' on its own line.
```

#### Random Track Selection
- **Checkbox**: Enable to use only a percentage of tracks from the playlist
- **Percent of Tracks**: What percentage to randomly select (1-100%)
- **Auto-Enabled**: Automatically enabled for playlists larger than the "similar tracks per track" value
- **Smart Default**: Percentage is calculated to yield approximately the same number of tracks as the "similar tracks per track" setting

**Use Case**: For large playlists (100+ tracks), sampling 20% can significantly reduce API costs while still providing good variety.

### API Call Counter

The blue info box shows estimated API calls before you generate:
- Updates in real-time as you adjust settings
- Accounts for random track selection percentage
- Helps estimate costs before running

**Example**: 
- 50 tracks in playlist
- Random selection: 20% = 10 tracks
- OpenAI + Last.FM both enabled
- **Result**: `10 OpenAI + 10 Last.FM` calls

## How It Works

### Step-by-Step Process

1. **Track Selection**: Loads tracks from the selected playlist, optionally randomly sampling if enabled
2. **API Requests**: For each track, requests similar tracks from enabled APIs:
   - **OpenAI**: Sends customized prompt to ChatGPT
   - **Last.FM**: Queries their track similarity database
3. **Candidate Collection**: Parses responses and builds list of recommended tracks
4. **Plex Search**: Searches your Plex library for each recommended track by artist + title
5. **Filtering**: 
   - Removes tracks already in the source playlist
   - Skips recently played tracks (based on "not played in X days")
   - Deduplicates across both API sources using canonical matching
6. **Script Generation**: Creates a Python script with all matched track IDs
7. **Download**: Automatically downloads the script to your computer

### Deduplication

Tracks are deduplicated using canonical matching:
- Artist name + track title normalized (lowercase, whitespace collapsed)
- Prevents the same song appearing twice from different sources
- Also prevents duplicate recommendations within the same API source

### Python Script Execution

The generated `createplaylist.py` script:
- Contains all configuration (server URL, token, playlist name, track IDs)
- Uses the PlexAPI library to connect to your server
- Fetches each track by its rating key
- Creates the new playlist with all discovered tracks
- Provides detailed output showing progress and results

## Output Log

The output log shows:
- Configuration summary
- Which tracks were queried
- API responses (OpenAI recommendations, Last.FM results)
- Plex search results for each candidate
- Reasons for skipping tracks (duplicates, recently played, not found)
- Final track list with rating keys
- Instructions for running the Python script

## Troubleshooting

### "Click Settings button to configure"
- You haven't entered your Plex server details yet
- Open Settings and fill in required fields

### "ERROR: Please select at least one track source"
- Both OpenAI and Last.FM checkboxes are unchecked
- Enable at least one API source

### "Last.FM selected but no API key configured"
- You enabled Last.FM but didn't provide an API key
- Add key in Settings or uncheck Last.FM

### OpenAI Errors
- **401 Unauthorized**: Invalid API key
- **429 Rate Limit**: Too many requests, wait and retry
- **500 Server Error**: OpenAI service issue, try again later

### No Tracks Found
- Recommendations might not exist in your library
- Try different source playlists with more mainstream artists
- Increase "similar tracks per track" value

### Script Won't Run
- Ensure PlexAPI is installed: `pip install PlexAPI`
- Verify Python 3 is installed: `python --version`
- Check that server URL and token in script are correct

## Privacy & Security

- **Local Storage Only**: All settings stored in browser localStorage
- **Client-Side Processing**: No data sent to third-party servers except APIs
- **API Keys**: OpenAI and Last.FM keys only used for direct API calls
- **No Server Required**: Runs entirely in your browser

⚠️ **Warning**: Don't share your browser with these settings or export/share the generated Python scripts, as they contain your Plex token.

## Tips for Best Results

1. **Start Small**: Test with smaller playlists (10-20 tracks) first
2. **Use Both APIs**: Combining OpenAI + Last.FM provides better coverage
3. **Tune Prompts**: Experiment with OpenAI prompts for different recommendation styles
4. **Random Sampling**: For playlists over 50 tracks, use 20-30% random selection
5. **Adjust Freshness**: Lower "not played" days if you want more track variety
6. **Check API Costs**: OpenAI charges per request - monitor the counter before generating

## Technical Details

- **Single File Application**: Everything in `index.html` (920 lines)
- **No Dependencies**: Pure vanilla JavaScript, no frameworks
- **Storage**: Browser localStorage for configuration persistence
- **APIs Used**:
  - Plex Media Server XML API
  - OpenAI Chat Completions API
  - Last.FM Track Similarity API

## License

This project is open source and available for personal use.
