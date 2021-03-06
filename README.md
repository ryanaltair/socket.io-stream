# socket.io-stream

socket.io based file stream

This package has three components

### Client:

  Only works in Nodejs environment like in ElectronJs.

### Web:

  Web is for browsers based app the component uses FileReader api to read file blob.

### Server:
  This component handlers all the request from both `Web` and `Client`

```js
//Client
import { Client } from '@ryanaltair/socket.io-stream'

const client = new Client(socket, {
  filepath: '/path/to/file/music.mp3',
  data: {
    //you pass your own data here
    name: 'music.mp3'
  }
})

client
  .upload('file-upload', data => {
    console.log({ data })
  })
  .on('progress', c => {
    console.log(c) //{total,size}
  })
  .on('done', data => {
    console.log(data)
  })
  .on('pause', () => {
    console.log('pause')
  })
  .on('cancel', () => {
    console.log('canceled')
  })
```

```js
//Web

function onChange(inputElement) {
  let file = inputElement.files[0]

  const client = new Web(socket, {
    file: file,
    data: {
      name: file.name
    }
  })
  client
    .upload('file-upload', data => {
      console.log({ data })
    })
    .on('progress', c => {
      console.log(c) //{total,size}
    })
    .on('done', data => {
      console.log(data)
    })
    .on('pause', () => {
      console.log('pause')
    })
    .on('cancel', () => {
      console.log('canceled')
    })
}
```

```js
//Server
import { Server } from '@ryanaltair/socket.io-stream'
import { appendFileSync } from 'fs'
const io = require('socket.io')(8090)

io.on('connection', socket => {
  const server = new Server(socket)

  server.on('file-upload', async ({ stream, data, ready }, ack) => {
    try {
      await someTask()
      ready()
      stream.subscribe({
        next({ buffer, flag }) {
          appendFileSync(data.name, buffer, {
            encoding: 'binary',
            flag
          })
        },
        complete() {
          ack(null, [1, 2, 3])
        },
        error(err) {
          ack(err)
          console.error(err)
        }
      })
    } catch (err) {
      console.error(err)
    }
  })
})
```