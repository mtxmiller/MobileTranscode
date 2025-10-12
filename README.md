# Mobile Transcode Rules

Optimized `custom-convert.conf` transcode rules for mobile Squeezebox clients.

## What This Plugin Does

This plugin provides optimized transcode rules for mobile streaming apps that connect to Lyrion Music Server (LMS). Once installed, LMS automatically uses these rules for supported mobile apps.

## Currently Supported Apps

- **LyrPlay** (iOS) - Optimized for iPhone/iPad streaming

## Installation

### For Users

**It's simple:**

1. In LMS web interface, go to **Settings → Plugins → Third Party**
2. Find **Mobile Transcode Rules** and click **Install**
3. **Restart LMS**
4. Done! Your mobile app automatically uses the optimized rules

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
flc flc YourAppName *
    # IFT:{START=--skip=%t}U:{END=--until=%v}D:{RESAMPLE=-r %d}
    [flac] -dcs $START$ $END$ --force-raw-format --sign=signed --endian=little -- $FILE$ | [sox] -q -t raw --encoding signed-integer -b $SAMPLESIZE$ -r $SAMPLERATE$ -c $CHANNELS$ -L - -t flac -r 44100 -C 0 -b 16 -

flc ops YourAppName *
    # IFT:{START=--skip=%t}U:{END=--until=%v}D:{RESAMPLE=-r %d}
    [flac] -dcs $START$ $END$ --force-raw-format --sign=signed --endian=little -- $FILE$ | [opusenc] --raw --raw-bits=$SAMPLESIZE$ --raw-rate=$SAMPLERATE$ --raw-chan=$CHANNELS$ --bitrate=256 - -

flc ogg YourAppName *
    # IFT:{START=--skip=%t}U:{END=--until=%v}D:{RESAMPLE=-r %d}
    [flac] -dcs $START$ $END$ --force-raw-format --sign=signed --endian=little -- $FILE$ | [sox] -q -t raw --encoding signed-integer -b $SAMPLESIZE$ -r $SAMPLERATE$ -c $CHANNELS$ -L - -t ogg -C 10 -
```

See `custom-convert.conf` in this plugin for a complete template with detailed comments.

#### Step 3: Test Thoroughly

- Seeking within tracks
- Track transitions
- Different FLAC bit depths (16-bit, 24-bit)
- Network conditions (WiFi, cellular)
- Check LMS logs for errors

#### Step 4: Submit Pull Request

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

### Capability Flags You Can Use

- **I** (Initial) - Synchronous pipeline launch (waits for transcoder)
- **F** (Fast) - Async pipeline launch (sends audio immediately)
- **T** (Time) - Supports seek with `{START=--skip=%t}`
- **U** (Until) - Supports duration with `{END=--until=%v}`
- **D** (Resample) - Supports resampling with `{RESAMPLE=-r %d}`

**Choose flags based on your app's requirements** - different platforms have different needs.

### Quality Parameters

**FLAC Output:**
- `-r 44100` or `-r 48000` - Sample rate (Hz)
- `-b 16` or `-b 24` - Bit depth (bits)
- `-C 0` to `-C 8` - Compression level (0=fast, 8=best)

**Opus Output:**
- `--bitrate=128` to `--bitrate=320` - Quality (kbps)

**OGG Vorbis Output:**
- `-C 0` to `-C 10` - Quality level

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

---

**Version**: 1.0.0
**Last Updated**: October 2025
