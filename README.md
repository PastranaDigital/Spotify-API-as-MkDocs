# Spotify API as MkDocs

This project is to convert the existing SPotify Web API documentation into MkDocs. During this effort I plan to better understand the abilities and limitations of MkDocs and its plug-ins.

This repository implements a GitHub workflow that publishes my changes to the GitHub page on merge.

<br><br>

## Links

Where it began: https://developer.spotify.com/documentation/web-api

My conversion: https://pastranadigital.github.io/Spotify-API-as-MkDocs/

<br><br>

## MkDocs Lessons Learned

1. Using custom stylesheets to control the visibility of "Table of Contents" for all pages
2. Controlling the order of the navgiation utilizing the "[Awesome Pages](https://github.com/lukasgeiter/mkdocs-awesome-pages-plugin?tab=readme-ov-file)" plugin
3. Linking between pages thru folders
4. Setting image width using `img` html code instead of `![Alt Text]` method
5. Compatibiltiy with Light or Dark mode using material toggle
6. Enabling site-wide search
7. Customized to use Logo and Favicon
