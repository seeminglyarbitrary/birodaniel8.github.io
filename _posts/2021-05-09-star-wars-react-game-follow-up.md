---
layout: post
title: "Star Wars React game follow-up"
author: "Daniel Biro"
categories: starwars
tags: [starwars, star wars, games, react, github, may4th, redux]
image: 2021-05-09-header-may4-follow-up.png
date: 2021-05-09
toc: true
---

Although my quite ambitious plan to put together my Star Wars game until May4th was just partly successful, 
the main functionalities were finished and I planned to continue the work on this little project.
It seems that after a couple of days of extra work, the game is up and running in its proper form. Check it out!

[Star Wars Card Game](https://birodaniel8.github.io/star_wars_react_game/){:target="_blank"}

The idea is still the same: you start a game with a randomly selected initial character card and the goal is to get to the target character card in the least amount of steps by clicking on attributes of the presented cards which then leads you to other cards. However, now the whole game is properly formatted, we have pictures and nice layouts. In the game mode, you see a maximum of 5 items per category but in the explore mode all items are presented to explore the whole galaxy. The game even has some extra help if you click on the target character's name, its picture will show up with an option to give up this round and jump to the target character. Also, the game works on mobile and tablet just as well as on desktop.

Used web development components:
 - **React** app with functional components (created by `create-react-app`)
 - **Redux** for cross-component state management
 - **Material-UI** provides the layout
 - **gh-pages** npm package for the GitHub Pages deployment
 - The data is from the **SWAPI**
 - The images are from **Wookiepedia** articles
 - **Python** is used to fetch and save the data from SWAPI* and to scrape and download the article pictures for the cards

The game's source code is available here: \
[https://github.com/birodaniel8/star_wars_react_game](https://github.com/birodaniel8/star_wars_react_game){:target="_blank"}

\* *I have decided to fetch and save all the data from the API as it is not too much to load, however, it is possible to fetch the data on the fly using JavaScript, but the number of requests to be sent is restricted and it made the development of the game cumbersome that way, however in the Git repo I included a file that shows how to do it.*

### Useful resources:
- [Understanding Functional Components vs. Class Components in React](https://www.twilio.com/blog/react-choose-functional-components){:target="_blank"}
- [Easy Redux Tutorial: Adding Redux to a Simple React App](https://www.youtube.com/watch?v=sX3KeP7v7Kg){:target="_blank"}
- [Deploying a React App to GitHub Pages](https://github.com/gitname/react-gh-pages){:target="_blank"}
- [The Star Wars API](https://swapi.dev/){:target="_blank"}
