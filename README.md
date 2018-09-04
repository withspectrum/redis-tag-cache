# `redis-tag-cache`

Cache and invalidate records in Redis with tags.

## Installation

```sh
yarn add redis-tag-cache
# or
npm install redis-tag-cache
```

## Demo Usage

```js
import TagCache from 'redis-tag-cache';

const cache = new TagCache();

/*
 * Cache some records tagged with IDs
 */ 

// Store two posts by the same author
await cache.set('post:id-123', { id: 'id-123', title: 'Hello world', author: 'user-123' }, ['id-123', 'user-123']);
await cache.set('post:id-234', { id: 'id-234', title: 'Hello world again', author: 'user-123' }, ['id-234', 'user-123']);
// And a third post by a different author
await cache.set('post:id-345', { id: 'id-345', title: 'Hello world again', author: 'user-234' }, ['id-345', 'user-234']);

/*
 * Retrieve records by their ID
 */

console.log(await cache.get('post:id-234')) // => { id: 'id-234', title: 'Hello world again', author: 'user-123' }

/*
 * Invalidate records by their tags
 */

// Invalidate all records tagged with `user-123`
await cache.invalidate(['user-123']);
console.log(await cache.get('post:id-123')) // => null
console.log(await cache.get('post:id-234')) // => null
// The third post not tagged with `user-123` is still around!
console.log(await cache.get('post:id-345')) // => { id: 'id-345', title: 'Hello world again', author: 'user-234' }
```

## API

### Base class

```JS
const cache = new TagCache(options, ioredisOptions);
```

#### `options`

Options can be an object containing any of the following keys:

- `defaultTimeout`: number of seconds until records expire even if not invalidated

Example:

```JS
const cache = new TagCache({
  defaultTimeout: 86400,
});
```

#### `ioredisOptions`

Any [`ioredis` option](https://github.com/luin/ioredis/blob/master/API.md#new-redisport-host-options), this object is directly passed through to `new Redis(ioredisOptions)`.

Example:

```JS
const cache = new TagCache(options, {
  keyPrefix: 'my-cache', // Recommended: set a keyprefix for all keys stored via the cache
  port: 6379,
  host: 'redis-service.com',
  password: 'password',
});
```

## License

Licensed under the MIT License, Copyright ©️ 2018 Maximilian Stoiber. See [LICENSE.md](LICENSE.md) for more information.
