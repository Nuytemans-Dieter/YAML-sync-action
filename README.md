# YAML-sync-action
This GitHub Action uses a template YAML file to sync changes to all other files.
It allows having multiple (language) config files and makes you only have to change one file in order to propagate these changes to the other files.
It is possible to add/remove config options or comments. When using this action, you only have to edit `template.yml` and commit the changes, this action will edit all other files accordingly. When adding new options, it is required to add this option to the default config file as well so that missing options in the other files can revert to the default. 

The repository must follow a few rules which have to be met:
- Restrictions
  - Any file to be synced may only contain String values
  - Any file to be synced may not contain multi-line values
  - All files to be synced must be in the same directory
  - The specified folder must not contain any file that should not be synced (eg. a config.yml file among localisation files. Folders or other extensions than .yml are allowed)
- Required files
  - A `template.yml` file is required. This file will be used to build all other files. For more info on this template file, follow the scenario below.
  - A default file is required to fill in any missing config options in the other files. When using this to sync localisation files, it is advised to use the most common (probably english) language file as default. The default file will be synced with the template as well. 

This action can take two inputs:
- `lang_path` which specifies the path to all language files. By default it will use the top-level of your repository.
- `default_file` specifies which file contains all default options and is used to fill in missing options in other files. It will be synced as well.

An example action step can be found below:
````
- name: Yaml autosync
  uses: Nuytemans-Dieter/YAML-sync-action@1.0.1
  with:
    lang_path: /example/path/to/folder
    default_file: default_file.yml
````

Let's examine a simple scenario below. Imagine a repo with following structure
````
| - .git/
| - src/
|    | - <redacted>
| - readme.md
| - resources
|    | - lang/
|         | - en-us.yml
|         | - template.yml
|         | - nl-be.yml
````

`template.yml` must contain all desired config options and comments as follows:
Note that we don't specify any default values in template.yml, it is merely a template to build our other files.
````
# This comment explains the setting below
some_lang_setting: {some_lang_setting}

# A random comment
# And another one!

another_setting: {another_setting}
````

Our default file must contain all options (or more) of template.yml. It does not matter which file is used as default, keep in mind that the default file is only used to fill in missing options of the other files. For this scenario, let's go with `en-us.yml`.
`en-us.yml` may look like this:
````
some_lang_setting: "This is the default of this setting"
another_setting: "This is the default of another setting"
````

The contents of all remaining files does not matter, as they will be completely overwritten. Let's assume that `nl-be.yml` contains a single value:
````
some_lang_setting: "Imagine a Dutch String here"
````

Our action setup would look like this:
````
- name: Yaml autosync
  uses: Nuytemans-Dieter/YAML-sync-action@1.0.1
  with:
    lang_path: /resources/lang/
    default_file: en-us.yml
````

Once we commit this change and enable the action, our files look like this:
Naturally, `template.yml` remains unchanged.

*en-us.yml*
````
# This comment explains the setting below
some_lang_setting: "This is the default of this setting"

# A random comment
# And another one!

another_setting: "This is the default of another setting"
````
*nl-be.yml*
````
# This comment explains the setting below
some_lang_setting: "Imagine a Dutch String here"

# A random comment
# And another one!

another_setting: "This is the default of another setting"
````
