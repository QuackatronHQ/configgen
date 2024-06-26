# analyze

## Requirements
- Docker Desktop 4.30.0
- GitHub CLI gh version 2.50.0
- wget (`brew install wget`)
- jq (`brew install jq`)

> Note: This current setup only supports arm64 architecture.

## Setup repository for analysis
- Create a `.deepsource.toml` file in the root of the repository. Ensure all values and metadata are configured properly in `.deepsource.toml` file for accurate analysis results. Refer to the [respective analyzer's docs](https://docs.deepsource.com/docs/analyzers) for more information. Refer to the `.deepsource.toml configuration file examples` section below for example configurations.

## Setup environment
- Authenticate `gh` with GitHub - `gh auth login`

- Create a Classis PAT with scope `read:packages` (Only Classic PATs are allowed; GitHub Container Registry does not support fine-grained access tokens yet)

- Create an env variable called `GH_PAT_DEEPSOURCE` with the GitHub PAT token
```sh
export GH_PAT_DEEPSOURCE=<PAT>
```
- Change directory to the repository root on which analysis is to be run
- Setup DeepSource analysis environment
```sh
wget https://raw.githubusercontent.com/QuackatronHQ/analyze/master/run.sh -O run.sh
chmod +x run.sh
./run.sh setup
```

## Run analysis
- From the current working directory as repository root, run the following command to start the analysis:
```sh
./run.sh analyze . <LANGUAGE>
```

> Note: Currently supported LANGUAGE values are: `go`, `python`, `java`, `ruby`, `javascript`, `docker`, `csharp`

- Once the analysis is complete, the results will be available in `analysis_results_<LANGUAGE>.json` and `analysis_results_<LANGUAGE>_pretty.json` files in the current working directory.

## Understanding the results
- The results file is JSON formatted and contains the issues in `issues` key. Each issue has the following keys:
    - `issue_code`: The unique identifier for the issue.
    - `issue_text`: The one-line description of the issue. Detailed description of each issue can be found in the [Analyzer Directory](https://app.deepsource.com/directory). For example: description of issue code `GO-2307` can be found at [app.deepsource.com/directory/analyzers/go/issues/GO-S2307](https://app.deepsource.com/directory/analyzers/go/issues/GO-S2307). DeepSource login not required.
    - `location`: The location of the issue in the code.
        - `path`: The path of the file where the issue is found.
        - `position`: The position of the issue in the file.
        - `begin`: The starting position of the issue.
            - `line`: The line number where the issue starts.
            - `column`: The column number where the issue starts.
        - `end`: The ending position of the issue.
            - `line`: The line number where the issue ends.
            - `column`: The column number where the issue ends.

## Limitations
- This setup supports analyzing only one language per repository at any given time. If you need to analyze multiple languages within the same repository, please conduct each analysis sequentially. Each analyzer's results are stored as `analysis_results_<LANGUAGE>.json` and `analysis_results_<LANGUAGE>_pretty.json` in the current working directory.
- Analysis is run on all files of the repository (excluding the patterns configured in `exclude_patterns`). Individual file analysis is not supported, unless it is the only file in the directory/repository.

## Cleanup
To cleanup all DeepSource generated files, run the following command:
```
./run cleanup
```
> Note: This will not cleanup the locally downloaded docker images.

## `.deepsource.toml` configuration file examples

### Python

```sh
version = 1

test_patterns = [
  "tests/**",
  "test_*.py"
]

exclude_patterns = [
  "migrations/**",
  "**/examples/**"
]

[[analyzers]]
name = "python"

  [analyzers.meta]
  runtime_version = "3.x.x"
```

### Go

```sh
version = 1

test_patterns = [
  "tests/*_test.go",
  "**/*_test.go"
]

[[analyzers]]
name = "go"
enabled = true

  [analyzers.meta]
  import_root = "github.com/DeepSourceCorp/marvin-go"
```

### Java

```sh
version = 1

test_patterns = [
  "test/**",
  "*Test.java"
]

[[analyzers]]
name = "java"

  [analyzers.meta]
  runtime_version = "21"
```

### Ruby

```sh
version = 1

test_patterns = [
  "test/**",
  "*_test.rb"
]

exclude_patterns = [
  "vendor/**",
  "**/examples/**"
]

[[analyzers]]
name = "ruby"
```

### JavaScript

```sh
version = 1

test_patterns = ["*/test/**"]

exclude_patterns = [
    "public/**,",
    "dist/**"
]

[[analyzers]]
name = "javascript"
```

### C#

```sh
version = 1

test_patterns = [
  "tests/**",
  "*Test.cs"
]

exclude_patterns = [
  "**/examples/**"
]

[[analyzers]]
name = "csharp"
```

### Docker

```sh
version = 1

[[analyzers]]
name = "docker"
```
