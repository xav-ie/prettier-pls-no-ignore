# Prettier Pls No Ignore 😱

According to this documentation, https://prettier.io/docs/en/cli#--ignore-path:

> --ignore-path

> Path to a file containing patterns that describe files to ignore. By default, Prettier looks for ./.gitignore and ./.prettierignore.
> Multiple values are accepted.

It _looks_ like Prettier is able to follow the .gitignore rules.

However, it does not! This is not mentioned ANYWHERE!

This is extremely frustrating.

This is also confusing because prettier has decided to follow the `.gitignore` syntax for their bespoke `.prettignore` file.

According to https://prettier.io/docs/en/ignore.html#ignoring-files-prettierignore:

<details>
<summary>"Prettier will also follow the fules specified in the '.gitignore' file if it exists in the same directory from which it is run." Click to expand for full quote from url.</summary>

> Ignoring Files: .prettierignore
>
> To exclude files from formatting, create a .prettierignore file in the root of your project. .prettierignore uses gitignore syntax.
>
> Example:
>
> # Ignore artifacts:
>
> build
> coverage
>
> # Ignore all HTML files:
>
> \*_/_.html
>
> It’s recommended to have a .prettierignore in your project! This way you can run prettier --write . to make sure that everything is formatted (without mangling files you don’t want, or choking on generated files). And – your editor will know which files not to format!
>
> By default prettier ignores files in version control systems directories (".git", ".sl", ".svn" and ".hg") and node_modules (unless the --with-node-modules CLI option is specified). Prettier will also follow rules specified in the ".gitignore" file if it exists in the same directory from which it is run.
>
> So by default it will be
>
> **/.git
> **/.svn
> **/.hg
> **/node_modules
>
> and
>
> **/.git
> **/.svn
> \*\*/.hg
>
> if --with-node-modules CLI option provided
>
> (See also the --ignore-path CLI option.)

</details>

However, this is _not true_! :(

In the first commit of this repo, it is simply contains just:

```
node_modules
```

Running `prettier --check .` returns that prettier wants to format `fake-dist/no-formatting-pls.js` and `fake-dist/pls-format-me.js`. This makes sense!

Second commit. We change .gitignore to contain:

```
node_modules
fake-dist
!fake-dist/*.json
```

Why would you do this? Well, for the sake of example pretend this was a generated file.

This is not uncommon for generated files to live in dist directories and be unignored for code review purposes. Think schema files, graphql files, and other types of files that are important to be committed for the review process. It is also often times important to format these files for easier review process.

Unfortunately, the documentation is not correct. Prettier will follow its made up rules of reading files which I (and probably you, too) do not know what they are, because, well, the documentaiton is incorrect and follows no convention whatsoever and does not explain itself.

## Workarounds

1. Use a different directory for generated files - Sure, I can concede that putting generated files in a `fake-dist` directory like this could be argued to just put in a different directory (i.e. `generated`), but there has been many times where I just don't have a choice. The files must live there in the same `fake-dist` directory. I have worked on many projects where this is also the case.
2. Ignore everything under the sun except for the files I want to format - This is a terrible idea. Just. Don't. This case may work:

```
node_modules
fake-dist/*.js
```

But what about when you want to not ignore some `.js` files and you have hundreds of `.js` files? You would have to list all the hundreds of `.js` files you don't want formatted except the ones you want. Oof.

3. Format all of `fake-dist` - No! If I want incremental builds, formatting my other files will not work. It breaks incremental building. It is also often just a lot of files I just don't care to format.
4. `.gitignore` hack - The documentation is correct in the fact that `.prettierignore` only reads from the local `./.gitignore` (yet another stray from convention 😢). A hack you could do is to put a `.gitignore` in the `fake-dist` folder and ignore normally. This way Prettier will be none-the-wiser since it only reads from `./.gitignore`. This is just not feasible unless I add a build step to emit a `./fake-dist/.gitignore` to my build steps because any full cleans should clear out my `fake-dist`.

All four of these workarounds are just bad. You have to admit that last one was pretty crafty :)!

## Moving forward

Prettier should follow the standard convention of allowing `!` operator because it is _useful_, _conventional_, and _expected_. If not, it should say it does actually follow `./.gitignore` rules when reading from the file. It just skips `!` rules.

However, not allowing `!` is not a good idea. The workarounds are are just awful. (open to new workaround ideas!)

By not allowing `!` we are just making Prettier more complicated than it has to be. Files that get committed should just get formatted. I really just can't think of an argument against, _"Well, files that get unignored are the special case and can never be formatted"_.

This is even harder to argue against given that the documentation says it follows `.gitignore` rules and the `.prettierignore` file uses the `.gitignore` syntax and even links to it in the documentation.

In fact, I think it would be better if Prettier had a way to handle sub-`.prettierignore` files, just like `.gitignore` recursive rules. This would open up a new workaround:

5. If your `fake-dist` directory happens to be in a package of a monorepo, you can just use the recursive feature of `.prettierignore` and format from the main repo. To elaborate, pretend this repo was a repo in a monorepo. A simple package with its own `./.gitignore`. The monorepo does not need to specify the files to ignore in this mini-package; the monorepo does not have a `./.gitignore`. This means it does not face this `./.gitignore` problem. It _can_ format your package because from the monorepo perspective, it is not ignored according to Prettier.

   Not so fast! We only want to format `./prettier-pls-no-ignore/fake-dist/*.json`. That's A-Ok! We can use workaround 2.... oh wait no. oh no. Workaround 2. kind of works when you have a limited file set, but the workaround explains why it is not feasible. For this reason, this workaround is given a half-feasible/10. It is also really confusing to have Prettier want to format a file on the monorepo level but not on the sub-repo level.

If Prettier had both recursive `.prettierignore` and followed `!` rules, this would solve a lot of problems for a lot of people.

There could be other soltions to this and I am happy to hear them, but I hope reading this you are at least opened to the idea that Prettier should have an opt-in option to follow all `.gitignore` behavior.
