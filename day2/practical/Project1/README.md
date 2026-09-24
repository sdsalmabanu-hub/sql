# Project1 Jenkins Demo

This is the sample repository content for Jenkins Day 2 practical.

## Goal

Jenkins will clone this repository, run `hello.sh`, and show the output in Console Output.

## Files

| File | Purpose |
| --- | --- |
| `README.md` | Explains the practical project |
| `hello.sh` | Script executed by Jenkins freestyle job |

## Jenkins Execute Shell

Use this in the Jenkins freestyle job:

```bash
echo "Project1 build started"
echo "Workspace: $(pwd)"
ls -la
chmod +x hello.sh
./hello.sh
echo "Project1 build finished"
```

Expected result:

```text
Finished: SUCCESS
```
