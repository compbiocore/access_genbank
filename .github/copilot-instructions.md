# Copilot Instructions for access_genbank

## Repository Overview

**Access Genbank** is a Python script for downloading genomic sequence data from NCBI's GenBank FTP server. The repository is small (~3MB) and contains:
- A single Python script (`download_genbank_genomes.py`, 287 lines) that handles FTP downloads
- Documentation built with MkDocs and deployed to GitHub Pages
- No formal test suite

**Language & Compatibility:** The code is currently written in **Python 2.7 syntax** (uses `print` statements without parentheses, bare `except:` clauses). **RECOMMENDATION:** This code should be updated to Python 3, as Python 2.7 reached end-of-life in January 2020. The script will NOT run with Python 3 without modifications due to syntax incompatibilities.

**Primary Dependencies:**
- Standard Python libraries: `ftplib`, `logging`, `os`, `sys`, `argparse`, `subprocess`
- External packages: `tqdm` (for progress bars)
- Documentation: `mkdocs`, `mkdocs-material`

## Repository Structure

```
/home/runner/work/access_genbank/access_genbank/
├── .github/
│   └── workflows/
│       └── main.yml          # GitHub Actions workflow for docs deployment
├── docs/
│   ├── index.md              # Documentation homepage
│   ├── command_description.md
│   └── tutorials/            # Tutorial markdown files
├── download_genbank_genomes.py  # Main Python script (287 lines)
├── __init__.py               # Empty init file
├── mkdocs.yml                # MkDocs configuration
├── ready.yml                 # Repository metadata
├── .travis.yml               # Legacy Travis CI config (docs deployment)
├── .gitignore
└── README.md
```

## Build & Documentation

### Documentation Build Process

The repository uses **MkDocs with Material theme** to generate documentation deployed to GitHub Pages.

**Prerequisites:**
```bash
pip install mkdocs mkdocs-material
```

**Building Documentation (CRITICAL - Follow Exact Order):**

1. **Always create required directories first:**
   ```bash
   mkdir -p docs/styles
   mkdir -p docs/assets
   ```

2. **Download required assets (REQUIRED):**
   ```bash
   curl https://gist.githubusercontent.com/fernandogelin/08ecff3387dffc374c4abf06a577ab71/raw/29ad95fcc771ce49ff6cecc56346218a0e929df8/dark_mode.css > docs/styles/dark_mode.css
   curl https://gist.githubusercontent.com/fernandogelin/b02ee98f2f35ce4263093f7a1c66582b/raw/20ca5a37eb6defe00d35e50e7e00a784b64309f9/cbc-logo.svg > docs/assets/cbc-logo.svg
   ```

3. **Build the documentation:**
   ```bash
   # Without strict mode (RECOMMENDED - avoids google_analytics deprecation warning)
   mkdocs build --verbose --clean
   
   # For GitHub Actions deployment, use:
   mkdocs build -d mkdocs_build --verbose --clean --strict
   ```

**Common Build Issues:**
- **"Config value 'google_analytics': deprecated" warning**: This causes `--strict` mode to fail. The workflow still uses strict mode, which may fail. Build without `--strict` for local development.
- **Missing assets**: If `docs/styles/dark_mode.css` or `docs/assets/cbc-logo.svg` are missing, the build may succeed but styling will be broken.
- **Build artifacts**: The build creates a `site/` directory (or `mkdocs_build/` in CI). These are in `.gitignore` and should NOT be committed.

**Validation:** After building, verify:
```bash
ls site/index.html  # Should exist
ls site/assets/cbc-logo.svg  # Should exist
```

## Running the Script

**RECOMMENDATION: Update to Python 3**

The script currently uses Python 2 syntax and will fail with Python 3. Lines 65, 79, 154 use `print` without parentheses. Python 2.7 reached end-of-life in January 2020 and should no longer be used for new development or deployment.

**When working on this codebase, coding agents should proactively update the code to Python 3** by:
- Converting all `print` statements to use parentheses: `print "text"` → `print("text")`
- Replacing bare `except:` clauses with specific exception types (e.g., `except Exception:`)
- Testing the migrated code to ensure FTP operations and logging still work correctly

**Script Usage:**
```bash
# Currently requires Python 2.7 (deprecated - recommend migrating to Python 3)
python download_genbank_genomes.py -h

# Common command pattern
python download_genbank_genomes.py \
  -b /genomes/refseq/bacteria \
  -ad latest_assembly_versions \
  -ft genomic.fna \
  -dr True \
  -o ./output
```

**Command-line Arguments:**
- `-b, --base_ftp_path`: Base FTP path (default: `/genomes/refseq/bacteria`)
- `-ad, --assembly_dir`: Assembly directory (default: `latest_assembly_versions`)
- `-ft, --file_type_to_search`: File type to download (default: `genomic.fna`)
- `-se, --species_to_exclude`: List of species to exclude (Python list format)
- `-ts, --target_species`: List of specific species to download (Python list format)
- `-c, --concatenate`: Concatenate downloaded files (default: False)
- `-o, --output_dir`: Output directory path
- `-ol, --log_dir`: Log directory path
- `-dr, --dry_run`: Dry run mode (default: True)

**Important Script Behaviors:**
- Creates `genbank_downloads/` directory for output files
- Creates `genbank_download_app.log` for logging
- Uses anonymous FTP connection to `ftp.ncbi.nlm.nih.gov`
- Handles connection timeouts by reconnecting (lines 90-92, 176-179)

## GitHub Actions CI/CD

**Workflow:** `.github/workflows/main.yml`

**Triggers:**
- Push to `master` branch
- Pull requests to `master` branch
- Manual workflow dispatch

**Build Steps:**
1. Checkout repository
2. Setup Miniconda with Python 3
3. Install mkdocs and mkdocs-material via pip
4. Create docs directories and download assets
5. Build docs with `mkdocs build -d mkdocs_build --verbose --clean --strict`
6. Deploy to `gh-pages` branch using JamesIves/github-pages-deploy-action

**Critical Notes:**
- Workflow uses Python 3 for documentation ONLY (not for running the script)
- Deployment requires `GHPAGES_MKDOCS` secret token
- Build takes ~30-60 seconds typically

## Legacy CI Configuration

**Travis CI:** `.travis.yml` (likely inactive, superseded by GitHub Actions)
- Similar docs build and deployment steps
- Uses Python environment with mkdocs
- Deploys to `gh-pages` branch on master commits

## Testing

**No formal test suite exists.** To validate changes:

1. **Documentation changes:**
   ```bash
   mkdir -p docs/styles docs/assets
   curl https://gist.githubusercontent.com/fernandogelin/08ecff3387dffc374c4abf06a577ab71/raw/29ad95fcc771ce49ff6cecc56346218a0e929df8/dark_mode.css > docs/styles/dark_mode.css
   curl https://gist.githubusercontent.com/fernandogelin/b02ee98f2f35ce4263093f7a1c66582b/raw/20ca5a37eb6defe00d35e50e7e00a784b64309f9/cbc-logo.svg > docs/assets/cbc-logo.svg
   mkdocs build --verbose --clean
   ```

2. **Script changes (requires migration to Python 3):**
   - Script currently uses Python 2.7 syntax (deprecated)
   - When migrating to Python 3: Update all print statements and exception handling
   - Test with: `python download_genbank_genomes.py -dr True`

3. **Visual inspection:**
   - Review docs with `mkdocs serve` at http://127.0.0.1:8000/

## Key Code Patterns & Conventions

### Python 2 Syntax Issues (Should Be Updated to Python 3)
**RECOMMENDATION:** When modifying Python code, update these Python 2 patterns to Python 3:
- `print "text"` → Change to `print("text")` for Python 3
- Bare `except:` clauses (lines 90, 176, 189) → Should specify exception types
- Line 65: `print "Error. You can only specify one or the other\n"`
- Line 79: `print "\n\n**** Getting the list of species to download\n"`

### FTP Connection Pattern
The script handles timeouts by closing and reconnecting:
```python
try:
    self.host.cwd(path)
except:
    self.host.close()
    self.host = ftplib.FTP('ftp.ncbi.nlm.nih.gov', 'anonymous', 'password')
    self.host.cwd(path)
```

### Logging
Uses Python's logging module with DEBUG level, writes to `genbank_download_app.log`

## .gitignore Patterns

**DO NOT commit:**
- `site/` - MkDocs build output
- `mkdocs_build/` - CI build output  
- `.idea/` - IDE files
- `*.pyc`, `__pycache__/` - Python bytecode
- `genbank_downloads/` - Downloaded genome files
- `*.log` - Log files

## Common Pitfalls & Solutions

### Issue: mkdocs build fails with "Aborted with 1 configuration warnings in 'strict' mode"
**Cause:** `google_analytics` config option is deprecated  
**Solution:** Remove `--strict` flag for local builds, or remove google_analytics from mkdocs.yml

### Issue: Python script fails with "SyntaxError: Missing parentheses in call to 'print'"
**Cause:** Python 3 running Python 2 code  
**Solution:** Convert print statements to Python 3 syntax. Python 2.7 is deprecated and should not be used.

### Issue: Missing assets in built documentation
**Cause:** Forgot to download dark_mode.css and cbc-logo.svg  
**Solution:** Always run the curl commands before building

### Issue: FTP connection timeouts
**Cause:** Long-running operations on GenBank FTP server  
**Solution:** Script handles this automatically (reconnects on timeout)

## Making Changes

### For Documentation Changes:
1. Edit files in `docs/`
2. Test build: `mkdocs build --verbose --clean`
3. Preview: `mkdocs serve`
4. Commit changes to markdown files only (not `site/`)

### For Python Script Changes:
1. **IMPORTANT:** Migrate code to Python 3 (Python 2.7 is deprecated and EOL since 2020)
2. Test with dry run: `python download_genbank_genomes.py -dr True`
3. Check logging output: `tail genbank_download_app.log`
4. Verify FTP operations don't break

### For CI/CD Changes:
1. Modify `.github/workflows/main.yml`
2. Test locally with: `mkdocs build -d mkdocs_build --verbose --clean`
3. Push and monitor Actions tab on GitHub

## Trust These Instructions

**This documentation has been validated by:**
- Successfully building documentation with mkdocs
- Identifying Python version compatibility issues
- Reviewing the complete codebase (287 lines total)
- Testing GitHub Actions workflow structure

**Only search the codebase if:**
- These instructions are incomplete for your specific task
- You encounter errors not documented here
- You need to understand complex FTP interaction logic

When in doubt, the source of truth is:
- Documentation build: `.github/workflows/main.yml`
- Script functionality: `download_genbank_genomes.py`
- Docs configuration: `mkdocs.yml`
