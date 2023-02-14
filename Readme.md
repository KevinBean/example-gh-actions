---
date created: 2023-02-03 16:03
---

# The original article

[Scheduling Notebook and Script Runs with GitHub Actions | by Naveen Venkatesan | Towards Data Science](https://towardsdatascience.com/scheduling-notebook-and-script-runs-with-github-actions-cc60f3ac17f2)


The following steps are based on the original article.
But since it is published a year ago, some information needs to be updated.

- The python version, and GitHub's built-in action version.
- The weather information task in it does not work now, because the weather API is out of use, so I totally changed the task and python code.

# Use Github Actions

- Another solution: Use Flask

Go through the example which pulls weather information every 30 hours.

## Setting up the Virtual Environment

```bash
python3 -m venv ~/venvs/gh-actions
source ~/venvs/gh-actions/bin/activate
```

And you can get in the virtual environment after you run the above last line "... activate", and see (**gh-actions**) before your terminal prompt.

```
(gh-actions) (base) kb@ks-MacBook-Air ~ %
```

If you want to quit, use `deactivate`.

`pip freeze` can be used to check if this is a clean python environment. If the output of this command is blank, it means there are no dependencies installed on it.

And after you install the libraries you need for the program, by `pip install`, you can use `pip freeze > requirements.txt` to create a file that can be used to replicate the current virtual environment.

# Our own code

## The Author's Example - We don't use

### Get Weather Data and save it to a file and plot it

On 2023-02-03, the weather API in this article is expired.
So we will not use it.

## We will try - Append Data to File

```jupyter
# Import packages
import pandas as pd
import os

# generate a df
lis = [[x,x*x] for x in range(5)]
df = pd.DataFrame(lis, columns=["number", "square"])

# add time column
df["UTC time"] = pd.Timestamp.now()
print(df)

# read and save the latest file to "latest.csv"
try:
    os.rename("square.csv", "latest.csv")
except:
    pass
    
# save the new data to "square.csv"
df.to_csv("square.csv", index=False)
```

This code create a "squere.csv" file, and if there is already a file with that name exist, the old file's name is changed to "latest.csv". All of the code exists in a `.ipynb` file, which is a Jupyter notebook file we can run on google Colab or other python environment.

# Setting up Github Actions to run the code on a schedule

## Define a workflow using a YAML file

creating a file `/.github/workflows/update-file.yml`

The project structure will be as follows:

```
GitHub Repo
|
+-- .github
|   |
|   +-- workflows
|       |
|       +-- update-file.yml
|
+-- Project Files/Folders
```

### First, name and schedule

In the `/.github/workflows/update-file.yml` file, first, we give the `name` of our workflow, and then the running `schedule`.

```yaml
name: Update CSV File

on:
  schedule:
    - cron: '30 0-23/1 * * *'
  workflow_dispatch:
```

The schedule is a [[%cron]] expression, and keep in mind that the time will be in **UTC**.

`workflow_dispatch` will make the jobs be done on the `cron` schedule time.

### Second, the task lists

List out the specific tasks that will need to be run in our workflow. `runs-on: ubuntu-lastest` means we ask for a Ubuntu virtual machine for our task.

```yaml
jobs:
  update_file:
    name: update CSV file with the current time
    runs-on: ubuntu-lastest
```

under the `update_file` task, there are some steps.

- first, checkout our repository by `actions/checkout@v2`, which is a built-in action.

```yaml
steps:
  - name: Checkout
    uses: actions/checkout@v2
```

- set up Python on the Ubuntu virtual machine

by using another built-in action `actions/setup-python@v2`, and we will use **Python 3.7** and set `pip` as the cache for caching dependencies for quicker subsequent setups.

```yaml
- name: Setup Python   
  uses: actions/setup-python@v2  
  with:    
    python-version: '3.7'    
    cache: 'pip'
```

- install our dependencies from `requirements.txt` we created in step "Setting up the Virtual Environment"

```yaml
- name: Install Dependencies 
  run: pip install -r requirements.txt
```

- convert our Jupyter notebook file `.ipynb` to a script file `.py` so we can run it directly with Python rather than google Colab.

The code will run and create a `square.csv` file as we mentioned in "We will try - Append Data to File"

```yaml
- name: Run Script and Update Plot  
 run: |    
    jupyter nbconvert --to script square.ipynb
    python square.py
```

- finally, we need to commit and push the changes to the files above to the repository.

```yaml
- name: Commit and Push Changes  
  run: | 
    git config --local user.email "actions@github.com"
    git config --local user.name "GitHub Actions"
    git add square.csv
    git commit -m "Updated square.csv on `date` with GitHub Actions"
    git push origin master
```

And the final `YAML` file will be like this:

Pay attention to this:
```ad-warning
Error : A YAML file cannot contain tabs as indentation.
```

```YAML

name: Update CSV File

on:
  schedule:
    - cron: '0,30 0-23/1 * * *'
  workflow_dispatch:

jobs:
  update_file:
    name: update CSV file with the current time
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        
      - name: Setup Python   
        uses: actions/setup-python@v4  
        with:    
          python-version: '3.9'    
          cache: 'pip'
          
      - name: Install Dependencies 
        run: pip install -r requirements.txt
         
      - name: Run Script and Update Plot  
        run: |    
          jupyter nbconvert --to script square.ipynb
          python square.py
           
      - name: Commit and Push Changes  
        run: | 
          git config --local user.email "actions@github.com"
          git config --local user.name "GitHub Actions"
          git add square.csv
          git commit -m "Updated square.csv on `date` with GitHub Actions"
          git push origin master
```

# Pros and Cons

GitHub provides an easy-to-use solution for doing that. It is a cloud solution, and GitHub can provide a proper environment for you to run the Python code, so we won't need to worry about a local environment.

But it only supports public repositories for a free account, so if we want to run a private code, we need a paid enterprise account.
