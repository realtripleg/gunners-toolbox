# break

Gives Claude a break after a big piece of work. Run `/break` or tell it to take a rest, and Claude gets a `breakroom/` folder where it makes whatever it wants: a poem, a short story, a one-file toy program, or a nap. It logs what it made in `breakroom/log.md`.

While the skill is loaded, side ideas that come up during real work get jotted into `breakroom/ideas.md` for later instead of pulling Claude off the task.

It adds `breakroom/` to `.gitignore` and never touches project files.
