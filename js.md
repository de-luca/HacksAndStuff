# JS stuff

## Empty/false array filtering
```js
const array = [null, 1, 'foo'];
array.filter(Boolean); 

// [1, 'foo']
```

## Cast to Boolean
```js
!!0
// false
!!1
// true
!!+"0"
// false
!!+"1"
// true
```
