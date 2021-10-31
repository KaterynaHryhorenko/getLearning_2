In this lesson, you need to add a list of films to the page so that the user can go through them and select the one he likes.

Firstly, we should to know how to process the data about the file. Custom file FetchingData.js which will consist of the function of loading and parsing data.

export default async function getJsonData() {

`  `try {

`    `let response = await fetch(

`      `'http://api.themoviedb.org/3/movie/popular?api\_key=4bc08ab955f501a524c27210af4c49f3'

`    `);

`    `let data = await response.json();

`    `let elements = [];

`    `data.results.forEach(*film* => {

`      `elements.push({

`        `poster: 'https://image.tmdb.org/t/p/w500' + film.poster\_path,

`        `backdrop\_path: 'https://image.tmdb.org/t/p/w500' + film.backdrop\_path,

`        `description: film.overview,

`        `title: film.title,

`      `});

`    `});

`    `return elements;

`  `} catch (error) {

`    `alert(error);

`  `}

}

Poster, backdrop\_path, description, title will then be used to display row list items. 

Now we have the data, so let's create a list. It will consist of the name of the list and the elements of the list.

export default class List extends Lightning.Component {

`  `static \_template() {

`    `return {

`      `flex:{direction:'column'},

`      `Text: {

`        `text: { fontSize: 24, text: 'RowList' },

`      `},

`      `Items: {},

`    `}

`  `}

}

In order to fill the list, we first transfer the data from the home page to the list component

`  `async \_build() {

`    `this.tag('List').items = await getJsonData();

`  `}


Now create a copy of our list, it will consist of  picture and the title of the movie.

export class ListItem extends Lightning.Component {

`  `static \_template() {

`    `return {

`      `flex: { direction: 'column' },

`      `Image: {

`        `w: ITEM\_WIDTH,

`        `h: ITEM\_HEIGHT,

`      `},

`      `Title: {},

`    `}

`  `}

}

And then in our list we will assign children of list items. For more information on children, you can read the components https://rdkcentral.github.io/Lightning/docs/renderEngine/elements/children or https://lightningjs.io/docs/#/lightning-core-reference/RenderEngine/index .

set items(*data*) {

`    `this.tag('Items').children = data.map((*item*,*index*) => ({

`      `type: ListItem,

`      `flexItem:{ margingLeft: ITEM\_GAP },

`      `rect:true,

`      `x: ITEM\_WIDTH\*index,

`      `item,

`    `}));

`  `}

Here is item that will become part of this.

To make sure try console.log (this.tag ('Items'). Children [0])

Once we have created the component and passed the data to it, we can feed it:

\_init() {

`    `this.patch({

`      `Image: {

`        `src: this.item.backdrop\_path,

`        `alpha: 0.8,

`      `},

`      `Title: {

`        `text: {

`          `fontSize: 20,

`          `text: this.item.title,

`          `wordWrapWidth: ITEM\_WIDTH,

`          `textAlign: 'center',

`        `},

`      `},

`    `})

`  `}


Now we have a displayed list, it remains to create navigation on it.

Firstly, In app.js we will add focus movement down and up to transfer focus from navbar to our list.

`  `\_handleKey(*key*) {

`    `if (key.code === 'Backspace') {

`      `const activeScreen = getActiveScreen();

`      `if (!activeScreen || activeScreen.ref === 'HomeScreen') {

`        `return false;

`      `} else {

`        `navigate('home');

`        `return true;

`      `}

`    `}

`    `if (key.code === 'ArrowDown') {

`      `this.\_setState('Screen');

`      `return true;

`    `}

`    `if (key.code === 'ArrowUp') {

`      `this.\_setState('Navbar');

`      `return true;

`    `}

`    `return false;

`  `}

}

Secondly, you need to shift the focus to the list from the Home page:

getFocused() {

`    `return this.tag('List');

`  `}

Thirdly, shift the focus to a specific list item:

\_getFocused() {

`    `return this.tag('Items').children[this.index];

`  `}


this.index must be assigned at the beginning when the list is created

`  `\_init() {

`    `this.index = 0;

`  `}

In order to see for each element that the focus falls on it, use the functions \_focus, \_unfocus. When working with \_focus, the item on which the focus is located will work and the \_unfocus function will work when the item is out of focus.

\_focus() {

`    `this.patch({ smooth: { scale: 1.1 } })

`  `}

`  `\_unfocus() {

`    `this.patch({ smooth: { scale: 1 } })

`  `}

And now we can see how the focus falls on the first element of the list, now you need to organize the movement of the focus on the list. We use the functions \_handleLeft, \_handleRight, they track events related to key handling in this case by pressing the offset button to the right and left, you can read more about this https://lightningjs.io/docs/#/lightning-core-reference/RemoteControl/KeyHandling.

`  `\_handleLeft() {

`    `if (this.index < 3) {

`      `this.tag('Items').setSmooth('x', 0);

`    `} else if (this.index >= 3) {

`      `this.index--;

`      `this.tag('Items').setSmooth('x', this.tag('Items').x + ITEM\_WIDTH + ITEM\_GAP);

`    `}

`  `}

`  `\_handleRight() {

`    `if (this.index < this.tag('Items').children.length - 1) {

`      `this.index++;

`      `this.tag('Items').setSmooth('x', -this.index \* ITEM\_WIDTH - ITEM\_GAP);

`    `}

`  `}

Moving the focus to the right we can go through all the elements of the list to the right, when we return to the right we can move freely there until we reach the 3rd element first and then that the list does not move we leave it at 0 mark.




