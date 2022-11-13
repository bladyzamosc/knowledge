### 1. Change author of all commits

`git filter-branch -f --env-filter "
GIT_AUTHOR_NAME='bladyzamosc'
GIT_AUTHOR_EMAIL='bladyzamosc@gmail.com'
GIT_COMMITTER_NAME='bladyzamosc'
GIT_COMMITTER_EMAIL='bladyzamosc@gmail.com'
" HEAD`

`git push --force`

