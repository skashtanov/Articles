# How to freeze all object properties?

What should we do if we want to make JS object immutable?

Maybe i have a solution: 

```jsx
function deepFreeze(obj) {
	const properties = Object.getOwnPropertyNames(obj);
	for (const property of properties) {
		let value = obj[property];
		if (value && typeof value === 'object') {
			obj[property] = deepFreeze(value);
		}
	}
	return Object.freeze(obj);
}

const person = deepFreeze({
	name: 'Jeffrey',
	address: {
		country: 'London',
		street: 'Baker'
	}
});

person.address.street = 'Carnaby'; // TypeError: Cannot assign to read only property
```

@Sergey Kashtanov 

August 15, 2021