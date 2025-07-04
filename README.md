# rclip - AI-Powered Command-Line Photo Search Tool
<!-- ALL-CONTRIBUTORS-BADGE:START - Do not remove or modify this section -->
[![All Contributors](https://img.shields.io/badge/all_contributors-3-orange.svg?style=flat-square)](#contributors-)
<!-- ALL-CONTRIBUTORS-BADGE:END -->

[[Blog]](https://mikhalevi.ch/rclip-an-ai-powered-command-line-photo-search-tool/) [[Demo on YouTube]](https://www.youtube.com/watch?v=tAJHXOkHidw) [[Paper]](https://www.thinkmind.org/index.php?view=article&articleid=content_2023_1_20_60011)

<div align="center">
  <img alt="rclip logo" src="https://raw.githubusercontent.com/yurijmikhalevich/rclip/main/resources/logo-transparent.png" width="600px" />
</div>

**rclip** is a command-line photo search tool powered by the awesome OpenAI's [CLIP](https://github.com/openai/CLIP) neural network.

## Installation

### Linux

```bash
sudo snap install rclip
```

<details>
  <summary>Alternative options (AppImage and <code>pip</code>)</summary>

  If your Linux distribution doesn't support [snap](https://snapcraft.io/rclip), you can use one of the alternative installation options:

  #### AppImage (self-contained x86_64 executable)

  On Linux x86_64, you can install **rclip** as a self-contained executable.

  1. Download the AppImage from the latest [release](https://github.com/yurijmikhalevich/rclip/releases).

  2. Execute the following commands:

  ```bash
  chmod +x <downloaded AppImage filename>
  sudo mv <downloaded AppImage filename> /usr/local/bin/rclip
  ```

  #### Using <code>pip</code>

  ```bash
  pip install --extra-index-url https://download.pytorch.org/whl/cpu rclip
  ```
</details>

### macOS

```bash
brew install yurijmikhalevich/tap/rclip
```

<details>
  <summary>Alternative option (<code>pip</code>)</summary>

  ```bash
  pip install rclip
  ```
</details>

### Windows

1. Download the "*.msi" from the latest [release](https://github.com/yurijmikhalevich/rclip/releases).
2. Install **rclip** by running the installer.

<details>
  <summary>Alternative option (<code>pip</code>)</summary>

  ```bash
  pip install rclip
  ```
</details>

## Usage

```bash
cd photos && rclip "search query"
```

<img alt="rclip usage demo" src="https://raw.githubusercontent.com/yurijmikhalevich/rclip/main/resources/rclip-usage.gif" width="640px" />

When you run **rclip** for the first time in a particular directory, it will extract features from the photos, which takes time. How long it will take depends on your CPU and the number of pictures you will search through. It took about a day to process 73 thousand photos on my NAS, which runs an old-ish Intel Celeron J3455, 7 minutes to index 50 thousand images on my MacBook with an M1 Max CPU, and three hours to process 1.28 million images on the same MacBook.

For a detailed demonstration, watch the video: https://www.youtube.com/watch?v=tAJHXOkHidw.

### Similar image search

You can use another image as a query by passing a file path or even an URL to the image file, and **rclip** will find the images most similar to the one you used as a query. If you are referencing a local image via a relative path, you **must** prefix it with `./`. For example:

```bash
cd photos && rclip ./cat.jpg

# or use URL
cd photos && rclip https://raw.githubusercontent.com/yurijmikhalevich/rclip/main/tests/e2e/images/cat.jpg
```

Check this video out for the image-to-image search demo: https://www.youtube.com/watch?v=1YQZKeCBxWM.

### Combining multiple queries

You can add and subtract image and text queries from each other; here are a few usage examples:

```bash
cd photos && rclip horse + stripes
cd photos && rclip apple - fruit
cd photos && rclip "./new york city.jpg" + night
cd photos && rclip "2:golden retriever" + "./swimming pool.jpg"
cd photos && rclip "./racing car.jpg" - "2:sports car" + "2:snow"
```

If you want to see how these queries perform when executed on the 1.28 million images ImageNet-1k dataset, check out the demo on YouTube: https://www.youtube.com/watch?v=MsTgYdOpgcQ.

### How do I preview the results?

If you are using either one of [iTerm2](https://iterm2.com/), [Konsole](https://konsole.kde.org/) (version 22.04 and higher), [wezterm](https://wezfurlong.org/wezterm/), [Mintty](https://mintty.github.io/), or [mlterm](https://mlterm.sourceforge.net/) all you need to do is pass `--preview` (or `-p`) argument to **rclip**:

```bash
rclip -p kitty
```

<details>
  <summary>Using a different terminal or viewer</summary>

  If you are using any other terminal or want to view the results in your viewer of choice, you can pass the output of **rclip** to it. For example, on Linux, the command from below will open top-5 results for "kitty" in your default image viewer:

  ```bash
  rclip -f -t 5 kitty | xargs -d '\n' -n 1 xdg-open
  ```

  The `-f` param or `--filepath-only` makes **rclip** print the file paths only, without scores or the header, which makes it ideal to use together with a custom viewer as in the example.
  
  I prefer to use **feh**'s thumbnail mode to preview multiple results:

  ```bash
  rclip -f -t 5 kitty | feh -f - -t
  ```
</details>

### Advanced Options

#### Custom Database Location

By default, rclip stores its database in your system's application data directory. You can specify a custom location:

```bash
# Use a custom database file
rclip --db-path /path/to/my/photos.db "sunset"

# Store database on an external drive
rclip --db-path /mnt/photos/rclip.db "vacation"
```

#### Search in Different Directories

Search for images in a specific directory without changing your current directory:

```bash
# Search in a different directory
rclip --search-dir /path/to/photos "cats"

# Search external drive
rclip --search-dir /mnt/backup/photos "birthday"
```

#### Index-Only Mode

Index images without performing a search - useful for pre-building the database:

```bash
# Just index the current directory
rclip --index-only

# Index a specific directory
rclip --index-only --search-dir /path/to/new/photos

# Index to a custom database
rclip --index-only --db-path /path/to/custom.db --search-dir /photos
```

#### Performance Tuning

For large image collections, you can increase the SQLite cache size to improve performance, especially during indexing:

```bash
# Use 64MB cache for faster indexing (default is 2MB)
rclip --index-only --db-cache-size 64

# Use 128MB cache when indexing millions of images
rclip --index-only --db-cache-size 128 --search-dir /massive/photo/library
```

Note: Increasing cache size primarily speeds up indexing operations. For searching, the default cache is usually sufficient unless you have millions of images in a single directory.

#### Combining Options

You can combine multiple options for complex workflows:

```bash
# Search photos on external drive with custom database
rclip --db-path /mnt/photos/rclip.db --search-dir /mnt/photos "sunset" --top 20

# Fast search with no indexing
rclip --no-indexing "flowers"
```

### Database Management

#### Merging Databases

The `rclip-db-merge` tool allows you to combine multiple rclip databases into one. This is useful when:
- You have indexed photos on different computers
- You want to combine databases from different photo collections
- You need to consolidate multiple partial indexes

Basic usage:
```bash
rclip-db-merge db1.db db2.db merged.db
```

Options:
- `-v, --verbose` - Show detailed progress information
- `--force` - Overwrite output file if it exists
- `--check-versions` - Check database compatibility without merging
- `--batch-size` - Number of rows to process per batch (default: 10000)
- `--commit-interval` - Number of rows between commits (default: 50000)
- `--cache-size` - Database cache size in MB (default: 64)
- `--mmap-size` - Memory-mapped I/O size in MB (default: 1024)

Examples:
```bash
# Check if databases are compatible
rclip-db-merge photo_db1.db photo_db2.db output.db --check-versions

# Merge with progress information
rclip-db-merge photo_db1.db photo_db2.db merged.db -v

# Merge large databases with optimized settings
rclip-db-merge large1.db large2.db output.db \
  --batch-size 20000 \
  --cache-size 256 \
  --mmap-size 4096

# Force overwrite existing output
rclip-db-merge db1.db db2.db existing.db --force
```

The tool handles conflicts intelligently:
- When the same image path exists in both databases, it keeps the newer version
- Deleted images remain marked as deleted
- The merge preserves all metadata including vectors

## Get help

https://github.com/yurijmikhalevich/rclip/discussions/new/choose

## Contributing

This repository follows the [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) standard.

### Running locally from the source code

To run **rclip** locally from the source code, you must have [Python](https://www.python.org/downloads/) and [Poetry](https://python-poetry.org/) installed.

Then do:
```bash
# clone the source code repository
git clone git@github.com:yurijmikhalevich/rclip.git

# install dependencies and rclip
cd rclip
poetry install

# activate the new poetry environment
poetry shell
```

If the poetry environment is active, you can use **rclip** locally, as described in the [Usage](#usage) section above.

## Contributors ✨

Thanks go to these wonderful people and organizations ([emoji key](https://allcontributors.org/docs/en/emoji-key)):

<!-- ALL-CONTRIBUTORS-LIST:START - Do not remove or modify this section -->
<!-- prettier-ignore-start -->
<!-- markdownlint-disable -->
<table>
  <tbody>
    <tr>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/ramayer"><img src="https://avatars.githubusercontent.com/u/72320?v=4?s=100" width="100px;" alt="ramayer"/><br /><sub><b>ramayer</b></sub></a><br /><a href="https://github.com/yurijmikhalevich/rclip/commits?author=ramayer" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://www.caphyon.com"><img src="https://avatars.githubusercontent.com/u/15829334?v=4?s=100" width="100px;" alt="Caphyon"/><br /><sub><b>Caphyon</b></sub></a><br /><a href="#infra-Caphyon" title="Infrastructure (Hosting, Build-Tools, etc)">🚇</a></td>
      <td align="center" valign="top" width="14.28%"><a href="http://abidkhan484.github.io"><img src="https://avatars.githubusercontent.com/u/15053047?v=4?s=100" width="100px;" alt="AbId KhAn"/><br /><sub><b>AbId KhAn</b></sub></a><br /><a href="https://github.com/yurijmikhalevich/rclip/commits?author=abidkhan484" title="Code">💻</a></td>
    </tr>
  </tbody>
</table>

<!-- markdownlint-restore -->
<!-- prettier-ignore-end -->

<!-- ALL-CONTRIBUTORS-LIST:END -->

Thanks to [Caphyon](https://github.com/Caphyon) and Advanced Installer team for generously supplying **rclip** project with the Professional Advanced Installer license for creating the Windows installer.

This project follows the [all-contributors](https://github.com/all-contributors/all-contributors) specification. Contributions of any kind are welcome!

## License

MIT
