# Datum Brain's Official Blog ![](https://travis-ci.com/datumbrain/datumbrain.github.io.svg?branch=master)

# How to Post

Get the latest from master
```
git pull origin master
```

Create a new branch with the following format.

```
post/whatever-post-title-is
```

## Creating New Post

New posts go in `_posts` directory. Every single post is actually a file of the format<br><br>

```
YYYY-MM-DD-whatever-post-title.md
```

Write your post in markdown syntax

## Testing Locally

To test what you have created locally. Run <br>

```
bundle exec jekyll serve
```

The application is accessible at `http://localhost:4000/`

## Publish

Once everything went as expected, push your branch and create a pull request. Feel free to add a reviewer.

## More

To learn more what goes in a post. Please refer to [jekyll docs](https://jekyllrb.com/docs/)
