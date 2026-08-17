```text
GitHub
│
├── app/
│   └── Flask code
│
├── Dockerfile
│
├── k8s/
│   └── deployment.yaml
│        │
│        └── image:
│             ECR/a1:5be7dedf...
│
└── .github/
    └── workflows/
        └── ci-cd.yml
```
```test
1. GitHub push
       ↓
2. GitHub Actions starts
       ↓
3. Docker image built              ✅
       ↓
4. Image pushed to ECR             ✅
       ↓
5. deployment.yaml updated         ✅
       ↓
6. Git commit created              ✅
       ↓
7. Git push to GitHub               ✅

 ${{ github.sha }} is not a Linux variable and it is not created by Docker.

It is a GitHub Actions context variable.

1. GitHub knows which commit triggered the workflow

Suppose you do:

git push origin main

and the commit is:

5be7dedf48531e155cb0fee4c3c7eeee110d3fbe

GitHub receives that commit.

Then GitHub starts your workflow because you have:

on:
  push:
    branches:
      - main

GitHub internally knows:

Repository:
murthychiluka/flask-eks-argocd


Branch:
main


Commit:
5be7dedf48531e155cb0fee4c3c7eeee110d3fbe

2. GitHub provides this information to the runner

GitHub Actions has predefined contexts/variables.

One of them is:

github.sha

It means:

The commit SHA that triggered this workflow.

So:

${{ github.sha }}

might evaluate to:

5be7dedf48531e155cb0fee4c3c7eeee110d3fbe
3. Then your command

You wrote:

docker build \
  -t $REGISTRY/$ECR_REPOSITORY:${{ github.sha }} \
  .

GitHub Actions processes:

${{ github.sha }}

before the shell executes the command.

For example:

docker build \
  -t $REGISTRY/$ECR_REPOSITORY:5be7dedf48531e155cb0fee4c3c7eeee110d3fbe \
  .

Then the shell expands:

$REGISTRY

and:

$ECR_REPOSITORY

Suppose:

REGISTRY = 044260498233.dkr.ecr.us-east-1.amazonaws.com
ECR_REPOSITORY = a1

The final command effectively becomes:

docker build \
  -t 044260498233.dkr.ecr.us-east-1.amazonaws.com/a1:5be7dedf48531e155cb0fee4c3c7eeee110d3fbe \
  .
4. Think about the two different syntaxes

This is particularly important:

${{ github.sha }}

is GitHub Actions expression syntax.

Whereas:

$REGISTRY

is Linux shell variable syntax.

So your command contains both:

docker build \
  -t $REGISTRY/$ECR_REPOSITORY:${{ github.sha }} \
  .

There are two systems involved:

GitHub Actions
      |
      | evaluates
      ↓
${{ github.sha }}
      |
      ↓
5be7dedf48531e155cb0fee4c3c7eeee110d3fbe
      |
      ↓
Shell receives command
      |
      | expands
      ↓
$REGISTRY
$ECR_REPOSITORY
*****************************
6. What does [skip ci] mean?

This is intended to tell CI systems:

Don't start another CI workflow because of this commit.

Why do we need this?

Because your workflow itself is doing:

GitHub Actions
      |
      ↓
Modify deployment.yaml
      |
      ↓
git commit
      |
      ↓
git push
      |
      ↓
GitHub sees another push
      |
      ↓
Workflow could start again

That can potentially create a loop.

So:

Original commit
      ↓
GitHub Actions
      ↓
Build image
      ↓
Update deployment.yaml
      ↓
Commit [skip ci]
      ↓
Push
      ↓
Don't trigger CI again

Note: [skip ci] is commonly recognized by GitHub Actions for push/PR workflows, but the exact behavior depends on the event/workflow context. Your if conditions also help prevent unnecessary jobs.
Working directory
      |
      | git add
      ↓
Staging area
      |
      | git commit
      ↓
Local Git repository
      |
      | git push
      ↓
GitHub
```
With |

If you have multiple commands:

- name: Install Dependencies
  run: |
    python --version
    pip --version
    pip install -r app/requirements.txt
```

```text
in order to access the UI of ArgoCD, you need to run the below command
kubectl edit svc argocd-server -n argocd  #enable Node Port or
kubectl patch svc argocd-server -n argocd \
  -p '{"spec":{"type":"NodePort"}}'
```
