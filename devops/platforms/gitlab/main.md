# GitLab

GitLab provides CI/CD functionality that can be leveraged with our projects. As in any CI/CD platform, the central idea is that of pipelines, which are specific configurations of tasks to be performed. For example, a pipeline might be configured to checkout a project on a slave server, then run a build command, then run unit tests, then run integration tests, then deploying the project to a different server. A specific run of a pipeline is called a job.

In GitLab we use GiLab Runner to run jobs for our pipelines. For regular GitLab repositories we have by default some shared runners available, and already active: to see them, we can go to Settings > CI/CD and expand Runners. Active runners have a green circle next to them.

To allow a repository to use runners, we need to add a `.gitlab-ci.yml` file to it:

```yaml
build-job:
  stage: build
  script:
    - echo "Hello, $GITLAB_USER_LOGIN!"

test-job1:
  stage: test
  script:
    - echo "This job tests something"

test-job2:
  stage: test
  script:
    - echo "This job tests something, but takes more time than test-job1."
    - echo "After the echo commands complete, it runs the sleep command for 20 seconds"
    - echo "which simulates a test that runs 20 seconds longer than test-job1"
    - sleep 20

deploy-prod:
  stage: deploy
  script:
    - echo "This job deploys something from the $CI_COMMIT_BRANCH branch."
```

We can use a Docker image to run the job:

```yaml
default:
  image: ruby:2.7.2
```

We can validate a `.gitlab-ci.yml` file with the [CI Lint tool](https://docs.gitlab.com/ee/ci/lint.html).

## Merge requests

If you add `WIP:` in front of the merge request title, GitLab will prevent the merge request from being merged until the `WIP:` is removed.

## References

- https://docs.gitlab.com/ee/ci/quick_start/
- https://docs.gitlab.com/ee/ci/yaml/README.html
- https://docs.gitlab.com/ee/ci/examples/README.html
