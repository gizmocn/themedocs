$ jekyll build
# => The current folder will be generated into ./_site

$ jekyll build --destination <destination>
# => The current folder will be generated into <destination>

$ jekyll build --source <source> --destination <destination>
# => The <source> folder will be generated into <destination>

$ jekyll build --watch
# => The current folder will be generated into ./_site,
#    watched for changes, and regenerated automatically.


if you want to publish to github:

1. in local themedocs project folder, run "jekyll build --destination docs"
2. add docs folder to git.
3. commit and push to github(remote)
4. in github.com, go settings of this project, in GitHub Pages, check the source is master branch/docs folder.
and maybe you need change custom domain to "docs.cattheme.com".
5. wait 10 minutes to check the result.


if you want to see in localhost, run:

jekyll serve

and check it at http://127.0.0.1:4000

=====================================
I have install html to pdf convert, the official is https://wkhtmltopdf.org/

command:

wkhtmltopdf http://127.0.0.1:4000/themes/rabel/ rabel.pdf







