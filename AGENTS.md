# Working on Tome

Validate with `scripts/ci`. Immediately commit all changes using Conventional Commits syntax, then deliver them through Tollgate: on master run `tg push-master --wait`; in a Tollgate worktree submit with `tg candidate HEAD`, approve, and wait for promotion. Tollgate owns pushes to remote master.
