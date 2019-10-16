### Things to Make Your Life Easier

Run the eslint auto-fixer before a commit. This takes a bit of time, but is less time then going back and fixing
something because you missed a semi.

```sh
echo 'gulp lint --fix' > .git/hooks/pre-commit
```
