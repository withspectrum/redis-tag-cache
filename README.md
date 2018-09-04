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

const cache = new TagCache({
  defaultTimeout: 86400 // Expire records after a day (even if they weren't invalidated)
});

/*
 * Cache some records tagged with IDs
 */ 

// Store two posts by the same author
await cache.set(
  'post:id-123',
  { id: 'id-123', title: 'Hello world', author: 'user-123' },
  ['id-123', 'user-123']
);
await cache.set(
  'post:id-234',
  { id: 'id-234', title: 'Hello world again', author: 'user-123' },
  ['id-234', 'user-123']
);
// And a third post by a different author
// and set a custom timeout for it
await cache.set(
  'post:id-345',
  { id: 'id-345', title: 'Hello world again', author: 'user-234' },
  ['id-345', 'user-234'],
  { timeout: 604800 /* Cache for a week */ }
);

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

### cache.set

Store a record in Redis. Usage:

```JS
cache.set(key: string, data: any, tags: Array<string>, options?: Object): Promise<void>
```

#### `options`

Can have one of the following keys:

- `timeout`: number of seconds until the record times out, overrides `defaultTimeout` option

#### Example

```JS
cache.set('some-key', 'some-value', ['some-tag'], { timeout: 123, })cache
  .then(() => console.log('Stored successfully!'))
```

### cache.get

Get a record from the cache. Usage:

```JS
cache.get(key: string): Promise<?data>
```

#### Example

```JS
cache.get('existing-key')
  .then(data => console.log('Got record!', data));

cache.get('not-existing-key')
  .then(data => console.log('data is null', data === null));
```

### cache.invalidate

Invalidate a set of tags and any records associated with them. Usage:

```JS
cache.invalidate(tags: Array<string>): Promise<void>
```

#### Example

```JS
cache.invalidate(['some-tag', 'some-other-tag'])
  .then(() => console.log('Tags invalidated successfully!'))
```

## License

Licensed under the MIT License, Copyright ©️ 2018 Maximilian Stoiber. See [LICENSE.md](LICENSE.md) for more information.
