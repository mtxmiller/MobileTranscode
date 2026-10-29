# Mobile Transcode Rules

Optimized `custom-convert.conf` transcode rules for mobile Squeezebox clients.

## What This Plugin Does

This plugin provides optimized transcode rules for mobile streaming apps that connect to Lyrion Music Server (LMS). Once installed, LMS automatically uses these rules for supported mobile apps.

## Currently Supported Apps

- **LyrPlay** (iOS) - Optimized for iPhone/iPad streaming
- Other apps: add yours (see For Developers)

## Installation

### For Users

1. In LMS web interface, go to **Settings → Plugins**
2. Under Third Party, find **Mobile Transcode Rules** and click **Install**
3. **Restart LMS**
4. Done! Your mobile app automatically uses the optimized rules

If the plugin isn't listed, add this repository at the bottom of the Plugins page: `https://raw.githubusercontent.com/mtxmiller/MobileTranscode/main/repo.xml`

**How it works:** LMS automatically searches all plugin directories for `custom-convert.conf` files. When you install this plugin, LMS finds and uses the rules immediately after restart.

### For Developers

Want to add your mobile app's rules to this plugin? Here's how:

#### Step 1: Set Your Model Name

In your SlimProto HELO capabilities:
```
Model=YourAppName,ModelName=Your App Display Name,...
```

#### Step 2: Add Your Rules

Fork this repository and add your app's rules to `custom-convert.conf`:

```
# YourAppName (Platform) - Brief description of your optimization choices
# Two copies per rule: FT for local files, then IT with NOSTART=I for piped sources
flc flc YourAppName *
    # FT:{START=--skip=%t}U:{END=--until=%v}D:{RESAMPLE=-r %d}
    [flac] -dcs $START$ $END$ --force-raw-format --sign=signed --endian=little -- $FILE$ | [sox] -q -t raw --encoding signed-integer -b $SAMPLESIZE$ -r $SAMPLERATE$ -c $CHANNELS$ -L - -t flac $RESAMPLE$ -C 0 -

flc flc YourAppName *
    # IT:{START=--skip=%t}U:{END=--until=%v}D:{RESAMPLE=-r %d}E:{NOSTART=I}
    [flac] -dcs $START$ $END$ --force-raw-format --sign=signed --endian=little -- $FILE$ | [sox] -q -t raw --encoding signed-integer -b $SAMPLESIZE$ -r $SAMPLERATE$ -c $CHANNELS$ -L - -t flac $RESAMPLE$ -C 0 -
```

See `custom-convert.conf` in this plugin for a complete template with detailed comments.

#### Step 3: Submit Pull Request

Include:
- App name and platform
- Explanation of your quality/bandwidth choices
- Platform-specific notes if any

## Technical Details

### How It Works

**Rule Matching:**
LMS matches transcode rules based on the `Model` field in your SlimProto HELO packet.

```
Model=LyrPlay  →  Matches: flc flc LyrPlay *
Model=YourApp  →  Matches: flc flc YourApp *
```

**Rule Format:**
```
[source] [dest] [model] [player]
    # [flags]:{[capabilities]}
    [transcode pipeline]
```

### Offline Copies (Downloads)

LyrPlay downloads the songs coming up so it can keep playing through a dead
spot. A download (`/music/<id>/download.<type>`) reaches LMS with no player,
so only rules for any player (`* *`) can convert it. A `* *` rule to `ops` or
`ogg` would also change what other players stream (squeezelite lists those
before `flc`, and LMS takes the first match when "prioritize native format"
is off). So the plugin converts downloads to two private types that no player
advertises (`custom-types.conf`):

- `lyo` - Opus 256k, same as the Opus stream rule
- `lyv` - Ogg Vorbis quality 10, same as the Ogg stream rule

Only a request for `download.lyo` / `download.lyv` uses them; streaming to any
player is unchanged.

### Capability Flags You Can Use

- **I** (Input) - Reads the source from stdin: streaming services (Qobuz, Tidal…) and remote URLs. Without it, LMS skips the rule for those sources and plays them in their native format
- **F** (File) - Reads a local file from your library
- **T** (Time) - Supports seek with `{START=--skip=%t}`
- **U** (Until) - Supports duration with `{END=--until=%v}`
- **D** (Resample) - Supports resampling with `{RESAMPLE=-r %d}`. Only declare it if the command uses `$RESAMPLE$` (or the encoder resamples by itself)
- **E:{NOSTART=I}** - Leaves out `$START$` when the input is piped

**Why two copies.** LMS offers a local file as I first, then F, and uses the first rule that accepts one of them. With a single `IFT` rule, local files are piped too. For a cue-sheet track LMS then starts the pipe at the track and also passes `--skip`, so the track comes out empty. Put an `FT` copy first (local files, exact `--skip`/`--until` on the file) and an `IT` copy with `E:{NOSTART=I}` second (piped sources). Same profile line twice is allowed; LMS tries them in order.

### Quality Parameters

**FLAC Output:**
- `-r 44100` or `-r 48000` - Sample rate (Hz)
- `-b 16` or `-b 24` - Bit depth (bits)
- `-C 0` to `-C 8` - Compression level (0=fast, 8=best)

**Opus Output:**
- `--bitrate=128` to `--bitrate=320` - Quality (kbps)

**OGG Vorbis Output:**
- `-C 0` to `-C 10` - Quality level

**MP3 Output:**
- `-b 128` to `-b 320` - Bitrate (kbps), 320 recommended for quality

### Advanced: Manual Rule Location

If you want to see or edit the rules directly:

1. Go to **Settings → Information** in LMS
2. Find your **Plugins folder** path
3. Navigate to `Plugins/MobileTranscode/custom-convert.conf`
4. Edit as needed
5. Restart LMS

**Note:** Changes will be overwritten when you update the plugin. For permanent custom rules, fork the plugin or maintain your own `custom-convert.conf` in your LMS prefs directory.

## Why Mobile Apps Need Custom Rules

Mobile streaming has unique requirements:

- **Network Variability** - Cellular vs WiFi bandwidth
- **Battery Constraints** - Efficient codecs save power
- **Platform Differences** - iOS, Android behave differently
- **User Behavior** - Mobile users seek/skip frequently

Custom transcode rules let each app optimize for its platform and use case.

## Contributing

We welcome contributions from all mobile Squeezebox developers!

**To contribute:**
1. Fork: https://github.com/mtxmiller/MobileTranscode
2. Add your app's rules to `custom-convert.conf`
3. Update this README with your app name
4. Submit a pull request

## Support

- **Plugin Issues**: https://github.com/mtxmiller/MobileTranscode/issues
- **LyrPlay App**: https://github.com/mtxmiller/LyrPlay
- **LMS Forums**: https://forums.lyrion.org

## License

MIT License - Free to use and modify.

## Credits

- **Eric Miller (mtxmiller)** - Plugin creator, LyrPlay developer
- **Lyrion Community** - For maintaining LMS
- **Mobile Squeezebox Developers** - For keeping the ecosystem alive
