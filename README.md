# MLOpsV6

This is a demonstration repository for teaching a simple MLOps CI/CD flow with
GitHub Actions and Azure ML.

The repo contains:

- Python training code in `data-science/src`
- Azure ML job definitions in `mlops/azureml/train`
- Environment definitions in `data-science/environment`
- A demo GitHub Actions workflow in `.github/workflows/mlops-demo.yml`

## What To Show Learners

Use this repo to teach three ideas:

1. A GitHub Actions workflow is YAML configuration stored in the repo.
2. A workflow runs automatically when code is pushed, or manually from the
   GitHub Actions tab.
3. A production pipeline is normally the same pattern with additional cloud
   authentication, deployment commands, and approval controls.

## GitHub Actions Workflow Role

The workflow file is:

```text
.github/workflows/mlops-demo.yml
```

Important parts to explain:

- `name`: The pipeline name learners see in the GitHub Actions UI.
- `on`: The events that trigger the pipeline. This demo runs on `push`,
  `pull_request`, and manual `workflow_dispatch`.
- `permissions`: The GitHub token permissions used by the workflow.
- `jobs`: Groups of work that run on GitHub-hosted machines.
- `runs-on`: The runner operating system, here `ubuntu-latest`.
- `steps`: The actual commands or reusable actions executed by the job.
- `needs`: Makes one job wait for another job to pass first.
- `if`: Adds a condition so the Azure ML submission only runs when requested.
- `secrets` and `vars`: Store production values outside the repository.

The demo pipeline has two jobs:

- `validate`: Checks out the repo, sets up Python, compiles the training scripts,
  and confirms the expected MLOps files exist.
- `azureml-training-pipeline`: Optional manual job that submits
  `mlops/azureml/train/pipeline.yml` to Azure ML. This requires Azure secrets and
  repository variables before it can run successfully.

## Run The Workflow Manually

1. Push this repo to GitHub.
2. Open the repository in GitHub.
3. Go to **Actions**.
4. Select **MLOps demo pipeline**.
5. Click **Run workflow**.
6. Keep `run_azureml_submission` as `false` for the first classroom run.
7. Open the running workflow and discuss each job and step with learners.

This first run demonstrates CI validation without needing Azure credentials.

## Show A Code Change Triggering The Pipeline

Use a small, low-risk code change so learners can focus on the Git and pipeline
flow.

Example:

```bash
git checkout -b demo/change-training-depth
```

Edit `mlops/azureml/train/pipeline.yml` and change the sweep search space:

```yaml
max_depth:
  type: choice
  values: [1, 3, 5, 10, 15]
```

Then push the change:

```bash
git status
git add mlops/azureml/train/pipeline.yml
git commit -m "Update training sweep depth"
git push origin demo/change-training-depth
```

On GitHub:

1. Open a pull request from `demo/change-training-depth` into `main`.
2. Show that the workflow starts because of the `pull_request` trigger.
3. Open the workflow logs and show the validation job.
4. Merge the pull request.
5. Show that the workflow starts again because of the `push` to `main`.

This demonstrates how code changes move from local development to GitHub and
automatically trigger CI.

## How Changes Reach Production

For production, explain the flow like this:

1. Developer changes code or pipeline YAML.
2. Developer pushes a branch to GitHub.
3. Pull request runs validation checks.
4. Reviewer approves the pull request.
5. Merge to `main` triggers the production pipeline.
6. The production job authenticates to Azure using GitHub secrets.
7. The pipeline submits `mlops/azureml/train/pipeline.yml` to Azure ML.
8. Azure ML prepares data, trains the model, runs the sweep, and registers the
   selected model.
9. A later deployment stage can promote the registered model to an online
   endpoint using the YAML files under `mlops/azureml/deploy`.

For a real production run, configure these GitHub repository settings:

- Secret: `AZURE_CREDENTIALS`
- Variable: `AZURE_RESOURCE_GROUP`
- Variable: `AZUREML_WORKSPACE`

Then run the workflow manually with:

```text
run_azureml_submission = true
```

Keep this manual in the classroom until the learners understand validation,
secrets, cloud authentication, and cost implications.

## Suggested Classroom Script

Start with the repository:

- Show `data-science/src/train.py` as the model training code.
- Show `mlops/azureml/train/pipeline.yml` as the ML pipeline definition.
- Show `.github/workflows/mlops-demo.yml` as the automation wrapper.

Then explain the boundary:

- Azure ML YAML defines what happens in the ML platform.
- GitHub Actions YAML defines when and how that ML platform command is executed.

Then run the action:

- Run the workflow manually with Azure submission disabled.
- Open the logs and map each log section back to the YAML file.

Then change the code:

- Change one training parameter in `pipeline.yml`.
- Commit and push the branch.
- Open a pull request.
- Watch the workflow trigger automatically.

End with production:

- Explain that production uses the same workflow structure.
- Add secrets and repository variables.
- Add approval environments if needed.
- Submit the Azure ML pipeline after validation passes.
