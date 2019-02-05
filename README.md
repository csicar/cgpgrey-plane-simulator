# cgpgrey-planesimulator

This is a Website with a simulation of CGPGreys recent Video about airplane boarding. You can select the boarding method with the select menu at the top of the page.

It is intended for trying out different boarding mrthods and seeing their effect on the boarding time.

The Website is made using grafics from the original Video and for the technical part VueJS and CSS transitions.

## Custom Sorting

You can create you own sorting system. Follow these instructions:
- Open the Developer Console for your Browser
- define a JavaScript function called `sortFunction`
- for example:
	```javascript
		sortFunction(personA, personB) {
			return personA.row - personB.row // personA.column is also available
		}
	```
- select `Custom` in dropdown menu on the page
- click `Run`

---------------------------

# Development

## Project setup
```
yarn install
```

### Compiles and hot-reloads for development
```
yarn run serve
```

### Compiles and minifies for production
```
yarn run build
```

### Run your tests
```
yarn run test
```

### Lints and fixes files
```
yarn run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
