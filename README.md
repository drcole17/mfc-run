# mfc-run

A wrapper around `./mfc.sh run` that adds run descriptions, metadata snapshots, and automatic archiving of simulation outputs.

## Installation

```bash
# Clone the repo
git clone https://github.com/<your-org>/mfc-run.git ~/.local/bin/mfc-run-repo

# Symlink onto PATH
ln -sf ~/.local/bin/mfc-run-repo/mfc-run ~/.local/bin/mfc-run
```

## Usage

```
mfc-run <case_path> [mfc_flags...]        Run simulation and archive results
mfc-run --archive-only <case_path> [flags...]  Archive existing output (skip mfc.sh)
mfc-run --list-runs <case_name>           List archived runs for a case
mfc-run --help                            Show help
```

### Examples

```bash
mfc-run examples/2D_recovering_ellipse/case.py -n 24 --gpu --clean
mfc-run examples/2D_recovering_ellipse -n 2 --clean
mfc-run --archive-only examples/2D_recovering_ellipse/case.py
mfc-run --list-runs 2D_recovering_ellipse
```

## Configuration

`mfc-run` is configured via environment variables or a config file. Precedence: **env vars > config file > auto-detect/defaults**.

### Environment variables

| Variable | Description | Default |
|---|---|---|
| `MFC_ROOT` | Path to MFC repository | Auto-detected by walking up from the case directory |
| `ARCHIVE_BASE` | Root directory for archived runs | `~/.local/share/mfc/runs` |

### Config file

Create `~/.config/mfc/run.conf`:

```
MFC_ROOT=/path/to/MFC
ARCHIVE_BASE=/path/to/archive
```

Lines starting with `#` are comments. Only `MFC_ROOT` and `ARCHIVE_BASE` are recognized.

### MFC root auto-detection

If `MFC_ROOT` is not set via env var or config file, `mfc-run` walks up the directory tree from the case file (and from the current working directory) looking for a directory containing both `mfc.sh` and `toolchain/util.sh`. This works when running cases from within the MFC tree, which is the common pattern.

## Archive structure

```
${ARCHIVE_BASE}/
  <case_name>/
    runs/
      2025-01-15_143022_my-description/
        run_info.json       # metadata (status, duration, git hash, parameters)
        case.py             # snapshot of case.py at run time
        case_params.json    # JSON parameter dump
        D/                  # moved output directories
        silo_hdf5/
        restart_data/
        p_all/
        postproc/           # copies of analysis scripts and plots
    failed/
      2025-01-15_150000_bad-run/
        ...
```

## Dependencies

- `bash` (4.0+)
- `python3`
- `rsync`
- `git` (for recording MFC commit hash)
