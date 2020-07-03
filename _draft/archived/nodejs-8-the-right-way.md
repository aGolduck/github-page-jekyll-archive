```js
it('should emit a message event from a single data event', done => {
  client.on('message', message => {
    assert.deepStrictEqual(message, {foo: 'bar'})
    done()
  })
  stream.emit('data', '{"foo": "bar"}\n')
})
```

- cheerio
- xmldom
- jsdom
- sax
- node-dir
- commander
- request
- zeromq
