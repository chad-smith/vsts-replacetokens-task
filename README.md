[![Donate](images/donate.png)](https://www.paypal.me/qetza/5)

# Replace Tokens task
Azure Pipelines extension that replace tokens in **text** files with variable values.

## Usage
If you are using the UI, add a new task, select **Replace Tokens** from the **Utility** category and configure it as needed:

![Replace Tokens parameters](images/task-parameters.png)

If your are using a YAML file, add a task with the following syntax:
```yaml
- task: qetza.replacetokens.replacetokens-task.replacetokens@3
  displayName: 'Replace tokens'
  inputs:
    targetFiles: |
      **/*.config
      **/*.json => outputs/*.json
```

**Note:** the task will only work on text files, if you need to replace token in archive file you will need to first extract the files and after archive them back.

### Parameters
The parameters of the task are described bellow, in parenthesis is the YAML name:

- **Root directory** (rootDirectory): the base directory for searching files. If not specified the default working directory will be used. _Default is empty string_
- **Target files** (targetFiles): the absolute or relative newline-separated paths to the files to replace tokens. Wildcards can be used (eg: `**\*.config` for all _.config_ files in all sub folders). _Default is `**/*.config`_
> **Syntax**: {file path}[ => {output path}]  
>
> - `web.config` will replace tokens in _web.config_ and update the file.
> - `web.tokenized.config => web.config` will replace tokens in _web.tokenized.config_ and save the result in _web.config_.
> - `config\web.tokenized.config => c:\config\web.config` will replace tokens in _config\web.tokenized.config_ and save the result in _c:\\config\web.config_.
>
> **Wildcard support**
> - `*.tokenized.config => *.config` will replace tokens in all _{filename}.tokenized.config_ target files and save the result in _{filename}.config_.
> - `**\*.tokenized.config => c:\tmp\*.config` will replace tokens in all _{filename}.tokenized.config_ target files and save the result in _c:\tmp\\{filename}.config_.
>
> Only the wildcard _*_ in the target file name will be used for replacement in the output.\
> Relative paths in the output pattern are relative to the target file path.\
>
> **Negative pattern**\
> If you want to use negative pattern in target file, use a semi-colon `;` to separate the including pattern and the negative patterns. When using output syntax, only the wildcard in the first pattern will be used for generating the output path.
> - `**\*.tokenized.config;!**\dev\*.config => c:\tmp\*.config` will replace tokens in all _{filename}.tokenized.config_ target files except those under a _dev_ directory and save the result in _c:\tmp\\{filename}.config_.

- **Files encoding** (encoding): the files encoding used for reading and writing. The 'auto' value will determine the encoding based on the Byte Order Mark (BOM) if present; otherwise it will use ascii. (allowed values: auto, ascii, utf-7, utf-8, utf-16le, utf-16be, win1252 and iso88591). _Default is `auto`_
- **Write unicode BOM** (writeBOM): if checked writes an unicode Byte Order Mark (BOM). _Default is `true`_
- **Escape type** (escapeType): specify how to escape variable values. Value `auto` uses the file extension (`.json` and `.xml`) to determine the escaping and `none` as fallback. _Default is `auto`_ (allowed values: auto, none, json, xml and custom)
- **Escape character** (escapeChar): when using `custom` escape type, the escape character to use when escaping characters in the variable values.
- **Characters to escape** (charsToEscape): when using `custom` escape type, characters in variable values to escape before replacing tokens.
- **Verbosity** (verbosity): specify the level of log verbosity. _Default is `normal`_ (note: error and system debug are always on) (allowed values: normal, detailed and off)
- **Action on missing variable** (actionOnMissing): specify the action to take on a missing variable. _Default is `warn`_
  - _silently continue_ (continue): the task will continue without displaying any message.
  - _log warning_ (warn): the task will continue but log a warning with the missing variable name.
  - _fail_ (fail): the task will fail and log the missing variable name.
- **Keep token for missing variable** (keepToken): if checked tokens with missing variables will not be replaced by empty string. _Default is `false`_
- **Action on no file processed** (actionOnNoFiles):  specify the action when no file was processed. _Default is `continue`_ (allowed values: continue, warn, fail)
- **Token pattern** (tokenPattern): specify the pattern of the tokens to search in the target files. _Default is `default`_ (allowed values: default, rm, octopus, azpipelines, doublebraces and custom)
- **Token prefix** (tokenPrefix): when using `custom` token pattern, the prefix of the tokens to search in the target files. _Default is `#{`_
- **Token suffix** (tokenSuffix): when using `custom` token pattern, the suffix of the tokens to search in the target files. _Default is `}#`_
- **Use legacy pattern** (useLegacyPattern): if checked whitespaces between the token prefix/suffix and the variable name are not ignored. _Default is `false`_
- **Use legacy empty/default feature** (useLegacyEmptyFeature): if check use the old empty & default values features. The new feature/code can now distinguish between an undefined variable and an defined empty variable without the use of a specific "empty" token (_Empty value_ parameter). _Default is `true`_
- **Empty value** (emptyValue): with legacy empty/default feature: the variable value that will be replaced with an empty string. _Default is `(empty)`_
- **Use default value** (useDefaultValue): with new empty/default feature: if check replace variable not found with a default value specified in _Default value_. _Default is `false`_
- **Default value** (defaultValue): the value to be used if a variable is not found. With legacy empty/default feature: do not set to disable default value feature with the legacy feature; to replace with an empty string set the default value to the _Empty value_. _Default is an empty string_
- **Enable transformations** (enableTransforms): if checked transformations can be applied on variable values.  _Default is `false`_. The following transformations are available:
  - _lower_: make variable value lower case. Example: `#{lower(MyVar)}#`
  - _upper_: make variable value upper case. Example: `#{upper(MyVar)}#`
  - _noescape_: disable variable value escaping. (this can be used if you want to inject raw JSON or XML for example). Example: `#{noescape(MyVar)}#`
  - _base64_: encode variable value in BASE64. Example `#{base64(MyVar)}#`
- **Transform prefix** (transformPrefix): The prefix between transform name and token name. _Default is `(`_.
- **Transform suffix** (transformSuffix): The suffix after the token name. _Default is `)`_.
- **Variable files (JSON or YAML)** (variableFiles): the absolute or relative comma or newline-separated paths to the files containing additional variables. Wildcards can be used (eg: `vars\**\*.json` for all _.json_ files in all sub folders of _vars_). YAML files **must have** the `.yml`or `.yaml` extension otherwise the file is treated as JSON. Variables declared in files overrides variables defined in the pipeline.
- **Inline variables (YAML syntax)** (inlineVariables): A YAML formatted string containing inline variables. Variables declared inline overrides variables defined in the pipeline and in files.
- **Variable separator** (variableSeparator): the separtor to use in variable names for nested objects and arrays in inline variables or variable files. _Default is `.`_. Example: `{ 'My': { 'Value': ['Hello World!'] } }` will create a variable _My.Value.0_ with the value _Hello World!_.
- **Enable recursion in values** (enableRecursion): if checked token replacement (with transformation) will be run on values.Example: "Say: #{value1}#" with value1 = "hello #{upper(value2)}#" and value2 = "world!" will result in "hello WORLD!". _Default is `false`_
- **Send anonymous usage telemetry** (enableTelemetry): if checked anonymous usage data (hashed collection and pipeline id, no file parameter values, no variable values) will be sent to the task author only to analyze task usage. _Default is `true`_

### Output variables
The task creates the following as output variables:
- **tokenReplacedCount**: the total number of tokens which were replaced by a variable.
- **tokenFoundCount**: the total number of of tokens which were found.
- **fileProcessedCount**: the total number of files which were processed.
- **transformExecutedCount**: the total number of transformations which were executed.
- **defaultValueCount**: the total number of default value used.

## Data/Telemetry
The Replace Tokens task for Azure Pipelines collects anonymous usage data and sends them to its author to help improve the product by default. If you don’t wish to send usage data, you can change your telemetry settings through _Send anonymous usage telemetry_ parameter or by setting a variable or environment variable `REPLACETOKENS_DISABLE_TELEMETRY` to `true`.

## Tips
If you want to use tokens in XML based configuration files to be replaced during deployment and also have those files usable for local development you can combine the [Replace Tokens task](https://marketplace.visualstudio.com/items?itemName=qetza.replacetokens) with the [XDT tranform task](https://marketplace.visualstudio.com/items?itemName=qetza.xdttransform):
- create an XDT transformation file containing your tokens
- setup your configuration file with local developement values
- at deployment time
  - inject your tokens in the configuration file by using your transformation file
  - replace tokens in your updated configuration file

## Release notes
**New in 4.4.1**
- Task **4.2.1**
  - Fix compatibility with node 5.10.1 ([#277](https://github.com/qetza/vsts-replacetokens-task/issues/277)).
- Task **3.10.1**
  - Fix compatibility with node 5.10.1 ([#277](https://github.com/qetza/vsts-replacetokens-task/issues/277)).

**New in 4.4.0**
- Task **5.1.0**
  - Add support for inline variables ([#252](https://github.com/qetza/vsts-replacetokens-task/issues/252)).
  - Add support for recursive token replacement in values ([#201](https://github.com/qetza/vsts-replacetokens-task/issues/201)).
  - Add optional reworked feature to simplify empty and default values (this is a **breaking change** if enabled as the old _Empty value_ is not used anymore but replaced by an empty variable declaration).
- Task **4.2.0**
  - Add support for inline variables ([#252](https://github.com/qetza/vsts-replacetokens-task/issues/252)).
  - Add support for recursive token replacement in values ([#201](https://github.com/qetza/vsts-replacetokens-task/issues/201)).
  - Add optional reworked feature to simplify empty and default values (this is a **breaking change** if enabled as the old _Empty value_ is not used anymore but replaced by an empty variable declaration).
- Task **3.10.0**
  - Add support for inline variables ([#252](https://github.com/qetza/vsts-replacetokens-task/issues/252)).
  - Add support for recursive token replacement in values ([#201](https://github.com/qetza/vsts-replacetokens-task/issues/201)).
  - Add optional reworked feature to simplify empty and default values (this is a **breaking change** if enabled as the old _Empty value_ is not used anymore but replaced by an empty variable declaration).

**New in 4.3.0**
- Add task **5.0.0**
  - **Breaking change**: Migrate task to Node10 execution handler needing agent `2.144.0` minimum ([#228](https://github.com/qetza/vsts-replacetokens-task/issues/228), [#230](https://github.com/qetza/vsts-replacetokens-task/issues/230)).

**New in 4.2.1**
- Task **4.1.1**
  - Revert migrate tasks to Node10 execution handler ([#233](https://github.com/qetza/vsts-replacetokens-task/issues/233)).
- Task **3.9.1**
  - Revert migrate tasks to Node10 execution handler ([#233](https://github.com/qetza/vsts-replacetokens-task/issues/233)).

**New in 4.2.0**
- Task **4.1.0**
  - Migrate tasks to Node10 execution handler ([#228](https://github.com/qetza/vsts-replacetokens-task/issues/228), [#230](https://github.com/qetza/vsts-replacetokens-task/issues/230)).
- Task **3.9.0**
  - Migrate tasks to Node10 execution handler ([#228](https://github.com/qetza/vsts-replacetokens-task/issues/228), [#230](https://github.com/qetza/vsts-replacetokens-task/issues/230)).

**New in 4.1.0**
- Task **4.0.1**
  - Promoted to release.
  - Add `base64` transform ([#163](https://github.com/qetza/vsts-replacetokens-task/issues/163)).
  - Add action on no file processed ([#210](https://github.com/qetza/vsts-replacetokens-task/issues/210)).
- Task **3.8.0**
  - Add `base64` transform ([#163](https://github.com/qetza/vsts-replacetokens-task/issues/163)).
  - Add action on no file processed ([#210](https://github.com/qetza/vsts-replacetokens-task/issues/210)).

**New in 4.0.0**
- Add support for multiple task versions.
- Add task 4.x (preview)
  - **Breaking change**: Add output variables ([#160](https://github.com/qetza/vsts-replacetokens-task/issues/160)). (some older version of TFS/Azure Pipelines doesn't support output variables when used in release pipelines)
  - **Breaking change**: Add dropdown parameter _Token pattern_ to select token pattern ([#131](https://github.com/qetza/vsts-replacetokens-task/issues/131)). (users with customized token pattern will need to manually select one or `custom`)

**New in 3.7.1**
- Fix issue on binary files ([#193](https://github.com/qetza/vsts-replacetokens-task/issues/193)).
- Rollback output variables ([#196](https://github.com/qetza/vsts-replacetokens-task/issues/196)).

**New in 3.7.0**
- Add output variables _tokenReplacedCount_, _tokenFoundCount_ and _fileProcessedCount_ ([#160](https://github.com/qetza/vsts-replacetokens-task/issues/160)).
- Add support for variable transformations with _Enable tranformations_ ([#96](https://github.com/qetza/vsts-replacetokens-task/issues/96)).
- Add default value for tokens not found with _Default value_ (contribution from [ClemensSutor](https://github.com/ClemensSutor)).
- Group log outputs in Azure Pipelines output.
- Add support for variables in external YAML files with `.yml` or `.yaml` extension ([#177](https://github.com/qetza/vsts-replacetokens-task/issues/177)).

**New in 3.6.0**
- Add parameter _Use legacy pattern_ with default value to `false`. 

**New in 3.5.2**
- Fix issue when token prefix present but not as a token prefix ([#149](https://github.com/qetza/vsts-replacetokens-task/issues/149)).

**New in 3.5.1**
- Fix issue when variable `System.ServerType` is not defined ([#147](https://github.com/qetza/vsts-replacetokens-task/issues/147)).

**New in 3.5.0**
- Add anonymous usage telemetry.
- Ignore spaces between token prefix/suffix and variable name ([#143](https://github.com/qetza/vsts-replacetokens-task/issues/143)).

**New in 3.4.1**
- Fix JSON escaping of slash `/` ([#138](https://github.com/qetza/vsts-replacetokens-task/issues/138))

**New in 3.4.0**
- Add summary in logs with number of tokens found and replaced ([#126](https://github.com/qetza/vsts-replacetokens-task/issues/126)).
- Add support for variables in external JSON files ([#113](https://github.com/qetza/vsts-replacetokens-task/issues/113)).

**New in 3.3.1**
- **Breaking change**: If you were using negative pattern you need to use the semi colon `;` as a separator instead of new-line in _Target files_.
- Fix negative pattern support ([#127](https://github.com/qetza/vsts-replacetokens-task/issues/127)).

**New in 3.3.0**
- Add support for custom output file and wildcard support ([#114](https://github.com/qetza/vsts-replacetokens-task/issues/114)).

**New in 3.2.2**
- Fix matching issue with directory ([#122](https://github.com/qetza/vsts-replacetokens-task/issues/122)).

**New in 3.2.1**
- Fix log issue with escaped secret values.

**New in 3.2.0**
- Switch to [jschardet](https://github.com/aadsm/jschardet) for encoding detection when selecting `auto` in _File encoding_ ([#99](https://github.com/qetza/vsts-replacetokens-task/issues/99)).
- Switch to [azure-pipelines-task-lib](https://github.com/Microsoft/azure-pipelines-task-lib) v2.8.0.
- Add `auto` to _Escape type_ and set it as default value.
- Move _Escape type_, _Escape character_ and _Characters to escape_ to the main paramters section for easier discoverability.

**New in 3.1.0**
- Add _Verbosity_ parameter to allow detail logs without using `system.debug`.

**New in 3.0.0**
- **Breaking change**: If you were using the character escaping feature you need to select `custom` in _Escape values type_ parameter.
- Add support to escape JSON in variable values (contributed by Justin Gould)
- Add support to escape XML in variable values (contributed by Justin Gould)
- Add `Windows 1252` and `ISO 8859-1` encoding to _File encoding_ ([#67](https://github.com/qetza/vsts-replacetokens-task/issues/67))

**New in 2.3.0**
- Add support to escape characters in variable values ([#52](https://github.com/qetza/vsts-replacetokens-task/issues/52))

**New in 2.2.1**
- Fix issue with backslash in default target files value on mac ([#50](https://github.com/qetza/vsts-replacetokens-task/issues/50))

**New in 2.2.0**
- Fix issue on file not found when using network paths ([#40](https://github.com/qetza/vsts-replacetokens-task/issues/40), [#41](https://github.com/qetza/vsts-replacetokens-task/issues/41)).

**New in 2.1.0**
- Add support for comma-separator in _Target files_ ([#35](https://github.com/qetza/vsts-replacetokens-task/issues/35)).
- Add _Empty value_ parameter to allow token replacement with an empty string ([#32](https://github.com/qetza/vsts-replacetokens-task/issues/32)).

**New in 2.0.2**
- Fix invalid file permissions after saving files.

**New in 2.0.0**
- **Breaking change**: Migrated code to typescript to support cross-platform agent. This change requires the use of an agent at least in version 2.105.0 which is **not compatible with TFS 2015**. If you need to install the task on TFS 2015, download the vsix from the repository: [https://github.com/qetza/vsts-replacetokens-task/releases/download/v1.4.1/qetza.replacetokens-1.4.1.vsix](https://github.com/qetza/vsts-replacetokens-task/releases/download/v1.4.1/qetza.replacetokens-1.4.1.vsix)
- **Breaking change**: _File encoding_ parameter is now used when reading and writing files. Previously it was only used when writing.
- **Breaking change**: _File encoding_ doesn't support 'utf-32' and 'utf-32 (big endian)' anymore.
- **Breaking change**: _Target files_ parameter now only uses the new line as a separator for multi-values (previously it used new-line and semi-colon).
- Removed required _Root directory_, an empty value is equivalent to $(System.DefaultWorkingDirectory).

**New in 1.4.1**
- Fix missing method issue with new xplat agent (2.104.1)

**New in 1.4.0**
- Add variables expansion in variable values.
- Escape token prefix and suffix in regex pattern.

**New in 1.3.1**
- Fix wrong encoding constructors parameters.

**New in 1.3.0**
- Replaced parameter _Fail on missing_ with _Action_ in _Missing variables_ group.
- Add _Keep token_ parameter in 'Missing variables' group.
- Fix issue on empty file.

**New in 1.2.0**
- Add _Root directory_ task parameter to configure file search root directory (contributed by Jesse Houwing).
- Update _Target files_ task parameter to support newline-separator (contributed by Jesse Houwing).