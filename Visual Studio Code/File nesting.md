In this article we will create a nesting of configuration files, which can be annoying to see all the time in our file system.

For example, in the following image we see that we have many configuration files in the root of our project:

![file-nesting-1.png](/assets/images/visual-studio-code/file-nesting-1.png)

First, we will go to: _File_ > _Preferences_ > _Settings_

Then look in the "Features" section and "Explorer".
We must check the option "File Nesting: Enabled".

![file-nesting-2.png](/assets/images/visual-studio-code/file-nesting-2.png)

We can create as many patterns as we want, in this case we are going to edit the item "package.json" because we want to see the package file and inside it all the configuration files.

We edit the value so that it looks like this:
```
yarn.lock, *.js, *.json, *.md, .babelrc, .editorconfig, .eslintignore, .eslintrc, .gitignore, .npmrc
```

![file-nesting-3.png](/assets/images/visual-studio-code/file-nesting-3.png)

In this way we are indicating some particular files (yarn.lock, .babelrc, .editorconfig, .eslintignore, .eslintrc, .gitignore, .npmrc) and on the other hand extensions (*.js, *.json, *.md), with this second mode, all files with extension .js, .json and .md will be nested independently of their name.

This is the result we will get:

![file-nesting-4.png](/assets/images/visual-studio-code/file-nesting-4.png)

Once we expand it, we will see it as follows:

![file-nesting-5.png](/assets/images/visual-studio-code/file-nesting-5.png)

It is important to mention that as long as this setting is marked as active, it will impact all projects opened with Visual Studio Code.
