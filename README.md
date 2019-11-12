<p align="center">
  <img src="https://user-images.githubusercontent.com/3791941/31036931-072760fe-a534-11e7-8cd7-0565bdc2727c.png" width="100" height="60">

  <h3 align="center">Workshop: Building Blocks: Creating your own GitHub Actions with JavaScript<br></h3>

  <p align="center">
    <a href="https://githubuniverse.com/">Universe Program</a>
  </p>
</p>

# Building Blocks: Creating Your Own GitHub Actions With JavaScript

## Before we begin
* **Required**: Sign up for GitHub. 😀
* **Required**: Sign up for the [public beta](https://github.com/features/actions) of GitHub Actions. The account that you use for this should be the one you plan on using in the workshop.
* **Helpful**: Familiarity with GitHub ([example training course](https://lab.github.com/githubtraining/introduction-to-github)), JavaScript, and GitHub Actions

## Workshop

Welcome! In this workshop, "Creating Your Own GitHub Actions With JavaScript", we'll be providing a conceptual overview of GitHub Actions and then making our very own Action using JavaScript. Follow along here, in the slides, or in person – however you learn best.

### Objectives
* Learn about the key features of GitHub Actions, like the secret store and matrix builds
* Learn about the difference between an action and a workflow
* Learn how to find actions for your workflow
* Build a JavaScript action
* Learn about best practices for your action
* Learn how to publish an action to the GitHub Marketplace

### Contents

1. 👋Welcome!

1. 🤔[Introducing GitHub Actions](#introducing-github-actions)

1. 🆚[Container Actions vs JavaScript Actions](#container-actions-vs-javascript-actions)

1. 💻[Hands-on with GitHub action development](#lets-build-a-gitHub-action-in-javascript)

1. 🍰[Improvements to this Action](#improvements-to-this-action)

1. 🚀[Best Practices](#best-practices)

1. ❓[Questions](#questions)

### Introducing GitHub Actions

[GitHub Actions](https://github.com/features/actions) is a new feature that allows you to customize your workflow on GitHub. Originally released in beta in 2018, the latest version includes powerful CI/CD primitives, a familiar YAML syntax, and the ability to run as a script or in a container!

We'll go over the main components of GitHub Actions you'll experience as a software developer, starting with *workflows*.

#### Workflows

Everyone uses GitHub a little differently. ***Workflows* let you codify useful processes to your liking**; for example, welcoming new contributors to your project, closing out stale issues, ensuring license uniformity, or testing and continuous integration and delivery. Workflows are YAML files that let you kick off a series of actions in one or more [jobs](https://help.github.com/en/articles/workflow-syntax-for-github-actions#jobs) when certain [events](https://help.github.com/en/articles/events-that-trigger-workflows) occur. They belong in a special directory in the repository you want the workflow to execute on: `.github/workflows`.

A workflow that installed JavaScript dependencies and ran some tests might look something like this:

```yaml
# .github/workflows/test.yml
on:
  push:
    branches:         # array of glob patterns matching against refs/heads. Optional; defaults to all
    - master          # triggers on pushes that contain changes in master
    - feature/*

name: Test

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v1       # this is an action
    - uses: actions/setup-node@v1     # this is another action
    - name: npm install, test         # this is a script 
      run: |
          npm install
          npm test
```

Or one that set up a weekly "standup" issue every Monday for a team:

```yaml
# .github/workflows/weekly-radar.yml
name: Weekly Radar

on:
  schedule:
  - cron: 0 12 * * 1

jobs:

  weekly_radar:
    name: Weekly Radar
    runs-on: ubuntu-latest
    steps:

    - name: weekly-radar
      uses: imjohnbo/weekly-radar@master
      with:
        assignees: "teammate1 teammate2"
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

You might still see references out there to the old workflow version file, `.github/main.workflow`, but these are deprecated. If you still have a `main.workflow` lying around, you can use [`actions/migrate`](https://github.com/actions/migrate) to take a first pass at migrating the HCL syntax to YAML. I've found it's pretty good.

The easiest way to start a new workflow is by navigating to the Actions tab in a repository and clicking "New workflow" (https://github.com/:owner/:repo/actions/new). You'll be presented with [starter templates](https://github.com/actions/starter-workflows/) to choose from in many different languages.

![image](https://user-images.githubusercontent.com/2993937/67030581-b4025080-f0dd-11e9-870a-18057a23097d.png)

Though we won't cover them deeply in this workshop, [workflows](https://help.github.com/en/articles/configuring-workflows) expose some other powerful features worth mentioning:
* Matrix builds including various failure strategies
* Path and event filtering
* Runners of any modern OS with extensive list of [pre-installed software](https://help.github.com/en/articles/software-in-virtual-environments-for-github-actions)
* Additional containers to host services for a job in a workflow

#### Actions

***Actions* are simply reusable units of code** (JavaScript or container) – for example, [stale](https://github.com/actions/stale) or [setup-node](https://github.com/actions/setup-node) – and are [referenced in workflows by the `uses` tag](https://help.github.com/en/articles/workflow-syntax-for-github-actions#jobsjob_idstepsuses). [Metadata](https://help.github.com/en/articles/metadata-syntax-for-github-actions) including name, entrypoint, inputs, and outputs live in a special top-level file called `action.yml`.

Any public repository on GitHub can be an action if it fits the criteria above (private repositories can *use* public actions but can currently only *serve* as actions to workflows in the same repository). Additionally, workflows can call containers on a registry like GitHub Package Registry or Docker Hub.

A Hello World action could be as simple as:

```yaml
# action.yml
name: 'Hello World'
description: 'Print greeting message'
author: 'GitHub'
inputs:
  greeting:
    description: 'Who to greet'
    default: 'world'
runs:
  using: 'node12'
  main: 'index.js'
```

```javascript
// index.js
const core = require('@actions/core'); // npm install this

async function run() {
  try { 
    const greeting = core.getInput('greeting');
    console.log(`Hello, ${greeting}!`);
  } 
  catch (error) {
    core.setFailed(error.message);
  }
}

run();
```

#### Actions tab

With these concepts in mind, let's head to GitHub and see it...in action 😉!

Navigate to [this repository](https://github.com/universeworkshops/building-blocks-first-workflow) with JavaScript project and tests. Instead of running `npm i && npm t` on your own computer, let's get GitHub Actions to do it.
e
Like we mentioned before, adding a new workflow to a repository can be as easy as navigating to [its Actions tab](https://github.com/universeworkshops/building-blocks-first-workflow/actions/new), and following the prompts to choose from among the starters.

![image](https://user-images.githubusercontent.com/2993937/67100247-4f9eca00-f18d-11e9-94f1-d6636bee6fa2.png)

Looks like the Node.js starter has what we need.

![image](https://user-images.githubusercontent.com/2993937/67100090-0d758880-f18d-11e9-8f05-24eee1feac7b.png)

Minutes later, we should have a successful build across several versions of Node.js:

![image](https://user-images.githubusercontent.com/2993937/67100450-adcbad00-f18d-11e9-88ef-b96ba70d43c1.png)

So, what just happened?
- We put a workflow YAML in `.github/workflows`, defining a `push` [trigger](https://help.github.com/en/articles/events-that-trigger-workflows) on any branch
- We said we wanted to [use Ubuntu](https://help.github.com/en/articles/workflow-syntax-for-github-actions#jobsjob_idruns-on) and defined several versions of Node.js to use later in a [build matrix](https://help.github.com/en/articles/workflow-syntax-for-github-actions#jobsjob_idstrategy)
- We used the `actions/checkout` action to `git pull` the repository onto the runner
- We used the `actions/setup-node` action
- We used `run` to execute some shell commands (`bash` [by default](https://help.github.com/en/articles/workflow-syntax-for-github-actions#using-a-specific-shell))
- The Actions tab showed us live, streaming logs with the ability to copy and paste individual lines

The primary way to discover actions is the [GitHub Marketplace](https://github.com/marketplace?type=actions) – of course, [GitHub searches](https://github.com/search?utf8=✓&q=action.yml) or Google searches could also help.

### Container Actions vs JavaScript Actions

Now let's actually *make* an action. This brings us to a fork in the road: container or script? 

|                     | JavaScript action                    | Container action |
|---------------------|--------------------------------------|------------------|
| Virtual environment | Linux, MacOS, Windows                | Linux            |
| Language            | Anything that compiles to JavaScript | any              |
| Speed               | ++                                   | +                |

There are advantages to using either type, but of course for this workshop, we'll be Creating a JavaScript Action™. 

In the interest of time, [we'll use](https://github.com/universeworkshops/building-blocks-first-action) the Hello World action from [before](#actions). After adding the `index.js` and `action.yml` files, we can go check out our greeting in the Actions tab:

![image](https://user-images.githubusercontent.com/2993937/67107042-add1aa00-f199-11e9-9c81-a7ef8c6d0ab9.png)

So, what just happened?
- We wrote a plain ol' JavaScript file, `index.js`, referencing a package that we had already `npm i`ed and committed to the repository in `node_modules`. This might look strange if you've been around the JavaScript block before – an explanation is coming.
- We defined metadata about our action in a specially named filed, `action.yml`.
- Our workflow ran as before.

### Let's build a GitHub Action in JavaScript
1. Create a new repository using [this template repository](https://github.com/universeworkshops/create-release/generate)
2. Clone the newly created repository
    - `git clone git@github.com:<username>/create-release.git`
3. Check out a new branch for our changes:
    - `git checkout -b <branch-name>`
4. Let's define our `action.yml`
    - Navigate to `<repo>/action.yml` and open this file in the editor of your choice
    - Edit lines `1` to `3` to update the `name`, `description`, and `author` of your action
    - Let's define some inputs
      - A required input for the name of the `tag` the user has pushed
      - A required input for the name of the `release` the action will create
      - An optional input for allowing `draft` releases
      - An optional input for allowing `prereleases` to be created
    - Ensure your inputs have a `description` that makes sense, and the proper attributes for `required` and `default` values as needed
    - Let's define some outputs to provide release data to other actions in the workflow, allowing you to persist data between workflow steps
      - An `id` to store the ID of the created release
      - A `url` for the `html_url` that users can navigate to in order to view the release
      - A `upload_url` for the URL that is used to upload assets to the release
    - Optionally, add a `branding` section. You can pick an icon (and it's color), following [these instructions](https://help.github.com/en/articles/metadata-syntax-for-github-actions#branding) for the proper format and available icons
    - Commit and push your changes to your repository
5. Let's define a `sample-workflow.yml`
    - Navigate to `<repo>/.github/workflows/sample-workflow.yml` and open this file in the editor of your choice
    - Decide which event(s) to trigger the workflow off of (for example: on push of tags matching a specific pattern)
    - Add step to check out your repository
    - Define a step to create the release using our action
      - Add an `id` attribute to this step so we can access outputs in additional steps later if needed
      - Our action requires some inputs, so let's provide them, using the same input names as the variables we defined in our `action.yml` above
        - Remember, a `tag name` and `release name` are required, while marking the release as a `prerelease` or `draft` is optional
    - Commit and push your changes to your repository
6. We're ready to code! Let's write the action that will do all the work for us!
    - Navigate to `<repo>/src/create-release.js` and open this file in the editor of your choice
    - Instantiate an authenticated [GitHub Client](https://github.com/actions/toolkit/tree/master/packages/github#usage) to make API calls on GitHub:
    ```.js
    const github = new GitHub(process.env.GITHUB_TOKEN);
    ```
    - Extract the owner and repository from the payload that triggered the action. This is within the `repo` object, nested inside `context`
    ```.js
    const { owner, repo } = context.repo;
    ```
    - Instantiate the variables needed for our [Create Release API call](https://developer.github.com/v3/repos/releases/#create-a-release), leveraging [core.getInput](https://github.com/actions/toolkit/tree/master/packages/core#inputsoutputs). Remember these input names are the ones you defined in your `action.yml` and were provided in your `sample-workflow.yml`
    ```.js
    const releaseName = core.getInput('release_name', { required: true }).replace('refs/tags/', '');
    const draft = core.getInput('draft', { required: false }) === 'true';
    const prerelease = core.getInput('prerelease', { required: false }) === 'true';
    const tagName = core.getInput('tag_name', { required: true });
    ```
    - Trim the `tagName` to the format needed for our API call, i.e. remove the 'refs/tags' portion of the string (from 'refs/tags/v1.10.15' to 'v1.10.15')
    ```.js
    const tag = tagName.replace('refs/tags/', '');
    ```
    - Now we can call the [Create Release endpoint](https://developer.github.com/v3/repos/releases/#create-a-release), using the [Octokit.rest documentation](https://octokit.github.io/rest.js/#octokit-routes-repos-create-release) to form our API call
    ```.js
    const createReleaseResponse = await github.repos.createRelease({
      owner,
      repo,
      tag_name: tag,
      name: releaseName,
      draft,
      prerelease
    });
    ```
    -  Get the `ID`, `html_url`, and `upload_url` for the created Release from the [response object](https://developer.github.com/v3/repos/releases/#response-4). These are the same outputs we defined in our `action.yml`, and can be referenced by future steps in the workflow if needed
    ```.js
    const {
      data: { id: releaseId, html_url: htmlUrl, upload_url: uploadUrl }
    } = createReleaseResponse;
    ```
    - [Set the output variables](https://github.com/actions/toolkit/tree/master/packages/core#inputsoutputs) for use by other actions
    ```.js
    core.setOutput('id', releaseId);
    core.setOutput('html_url', htmlUrl);
    core.setOutput('upload_url', uploadUrl);
    ```
    - Commit and push your changes to your repository
      - `git add .`
      - `git commit -m "A great commit message"`
      - `git push --set-upstream origin <your-branch>`

### Improvements to this action
Now that you've had some hands on experience with this action, how would you improve it? Maybe you can think of additional actions that could be written and used along with this action. Let's break out into sessions for about 20 minutes to work on these ideas. Feel free to ask one of the staff members for assistance.

To get you started, here are a few example ideas:
- Edit the release body, either through a new action or exposing the `body` as an input
- Upload release assets to the existing release, or build a new action to do this
- Modify the `README.md` file to include the latest version of your action
- Add additional outputs, such as the `tag` that was used to create the release, for other actions to consume in other workflow steps

### Best practices
When writing an action, there are a few things to keep in mind that can help both your development experience, as well as your users' experience:
- Prefer bite-sized, "chainable", primitive actions over complicated actions that try to solve all use cases at once
  - This also allows you to troubleshoot a specific step in a workflow more easily
  - Remember the power of actions is in workflows that can consume multiple actions. For example, one could have multiple actions in a workflow that:
    1. Checks out the code
    1. Runs linting
    1. Builds the project
    1. Runs tests
    1. Creates a draft release
    1. Generates release notes
    1. Edits the created release to add these release notes
    1. Uploads build assets to the created release
    1. Publishes the draft release
    1. Sends a tweet that you have a new release

#### Example of a monolothic approach

```.yml
on: push

name: Build and Release

jobs:
  build_and_release:
    name: Build and Release
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v1.0.0
      - name: Build and Test
        uses: actions/build-and-test@v1.0.0
      - name: Create Release and Tweet
        id: create_release
        uses: actions/create-release@v1.0.0
        env:
          # Access the `GITHUB_TOKEN` secret from the repository's secrets
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          # Access the `TWITTER_TOKEN` secret from the repository's secrets
          TWITTER_TOKEN: ${{ secrets.TWITTER_TOKEN }}
        with:
          # Access the `ref` from the `github` payload object
          tag_name: ${{ github.ref }}
          release_name: Release ${{ github.ref }}
          prerelease: false
          # Access the `upload_url` output, using the `id` from the prior step who's outputs you want
          upload_url: ${{ steps.create_draft_release.outputs.upload_url }}
```

#### Example of a chainable approach

```.yml
on: push

name: Build and Release

jobs:
  build_and_release:
    name: Build and Release
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v1.0.0
      - name: Lint code
        uses: actions/linting@v1.0.0
      - name: Build project
        run: |
          npm build
      - name: Run tests
        run: |
          npm test
      - name: Create draft release
        id: create_draft_release
        uses: actions/create-draft-release@v1.0.0
        env:
          # Access the `GITHUB_TOKEN` secret from the repository's secrets
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          # Access the `ref` from the `github` payload object
          tag_name: ${{ github.ref }}
          release_name: Release ${{ github.ref }}
          draft: true
          prerelease: false
      - name: Generate release notes
        id: generate_release_notes
        uses: actions/generate-release-notes@v1.0.0
        env:
          # Access the `GITHUB_TOKEN` secret from the repository's secrets
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      - name: Edit release to add notes
        uses: actions/add-release-notes@v1.0.0
        env:
          # Access the `GITHUB_TOKEN` secret from the repository's secrets
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          # Access the `release_notes` and `release_id` outputs, using the `id` from the prior step who's outputs you want
          release_notes: ${{ steps.generate_release_notes.outputs.release_notes }}
          release_id: ${{ steps.create_draft_release.outputs.release_id }}
      - name: Upload release asset
        id: upload-release-asset
        uses: actions/upload-release-asset@v1.0.1
        env:
          # Access the `GITHUB_TOKEN` secret from the repository's secrets
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          # Access the `upload_url` output, using the `id` from the prior step who's outputs you want
          upload_url: ${{ steps.create_draft_release.outputs.upload_url }}
          asset_path: ./my-artifact.zip
          asset_name: my-artifact.zip
          asset_content_type: application/zip
      - name: Publish draft release
        uses: actions/public-draft-release@v1.0.0
        env:
          # Access the `GITHUB_TOKEN` secret from the repository's secrets
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          # Access the `release_id` output, using the `id` from the prior step who's outputs you want
          release_id: ${{ steps.create_draft_release.outputs.release_id }}
      - name: Send tweet about new release
        uses: actions/tweet-release@v1.0.0
        env:
          # Access the `TWITTER_TOKEN` secret from the repository's secrets
          TWITTER_TOKEN: ${{ secrets.TWITTER_TOKEN }}
        with:
          # Access the `upload_url` output, using the `id` from the prior step who's outputs you want
          upload_url: ${{ steps.create_draft_release.outputs.upload_url }}
```

- [Tagging your releases](https://help.github.com/en/articles/about-actions#versioning-your-action) allows you to not only version your changes, but allows your users to ensure their workflows do not break if a new version introduces changes they didn't expect
- Well-written documentation goes a long way in allowing your users to self-service and get started quickly with your action
  - A `README.md` should contain, at a minimum:
    - A list of any pre-requisites in order for your action to run
    - Inputs and Outputs that are listed in your `action.yml` with a definition of what they are and the format your action expects them in (i.e. a date format, string, integer, etc)
    - Example workflow file that showcases how someone can get started using your action right away. Preferably one that can be copy/pasted directly into their own repository for an immediate successful action run with minimal (if any) changes
  - A `CONTRIBUTING.md` provides a way for open source contributors to submit their own enhancements for your repository
    - State how to run the tests
    - Explain the release and versioning process
    - Call out specific steps to contribute (i.e. fork, submit PRs, add tests when necessary, etc.)
  - A proper [LICENSE file](https://help.github.com/en/articles/licensing-a-repository) that covers your repository, how you want the action to be used and shared, and how you'd like forks of your repository to be used
  - Adding a few simple tests can go a long way to check the basic functionality of your action
    - GitHub supports adding [build badges](https://help.github.com/en/articles/configuring-a-workflow#adding-a-workflow-status-badge-to-your-repository) to your repository by adding it to your `README.md`. This will inform users that your action's latest build is stable!
  - Your `action.yml` file should contain all of the [metadata](https://help.github.com/en/articles/metadata-syntax-for-github-actions) about your action, including a definition of any Inputs and Outputs
    - Place your `action.yml` in the root of your action's repository
    - Remember this metadata is used in the GitHub Marketplace, if you choose to submit your action to the marketplace
    - Consider adding a [`branding:` section](https://help.github.com/en/articles/metadata-syntax-for-github-actions#branding) to your `action.yml` to customize your marketplace listing
  - Publish your action to the GitHub Marketplace
    - Publishing your action is the best way to increase discoverability of your action to other users
    - Once published, the listing will automatically render changes from your `README.md` to the marketplace listing page
    - Additionally, any new releases you tag in your repository will automatically be available in the marketplace listing page if you desire

### Questions
- Q: When will there be support for GitHub Enterprise?
  - A: GitHub Enterprise Cloud is already available, and [GitHub Enterprise Server](https://github.com/enterprise) support is a priority for the engineering team. A specific release timeline is not currently available.

## Resources

* [Slides](#TBD-public_slides)
* [@imjohnbo](https://github.com/imjohnbo)
* [@iamhughes](https://github.com/iamhughes)
