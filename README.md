# Test Chat Rails
Esencialmente Django es una mierda por que no facilita las API REST, ni tiene un buen scaffold al no ser homoiconico no permite un buen manejo de metaprogramacion... pero era dificil hacer RealTime Apps en Rails hasta que crearon ActionCable, si esto funciona sera un buen backend para React:

**usar RVM:**

http://railsapps.github.io/installrubyonrails-ubuntu.html

si hay alguno de los siguientes errores:
```
Webpacker configuration file not found
...

Please run rails webpacker:install Error: No such file or directory @ rb_sysopen .../config/webpacker.yml
...
RAILS_ENV=development environment is not defined in config/webpacker.yml, falling back to production environment
```

Instalar Yarn y actualizar:
```
rails webpacker:install
```


ya con uno creado si no esta activo, (no se si va con el create):
```
/bin/bash --login
rvm use ruby@rails --create
```

## Proyecto

Crear:
```
rails new actioncable_app
cd actioncable_app
rails db:create
```
Y creamos los modelos, controladores y el Channel:
```shell
rails g model Message body:string
rails g controller Messages
rails g channel Chat
rails db:migrate
```
Si usamos coffescript borrar el `chat.coffee` por que usaremos React

Editamos el `chat_channel.rb` para manejar la logica del socket, solo se pueden pasar objetos via Stream, por eso se usa el hash:
```ruby
class ChatChannel < ApplicationCable::Channel
  def subscribed
    stream_for 'chat_channel'
  end
  def speak(data)
    message = Message.create(body: data['message'])
    socket = { message: message.body }
    ChatChannel.broadcast_to('chat_channel', socket)
  end
  def unsubscribed; end
end
```

En `app/views/messages` creamos un archivo `root.html.erb`:
```html
<head>
    <!-- Required meta tags -->
    <meta charset="utf-8">
    <meta
      name="viewport"
       content="width=device-width, initial-scale=1, shrink-to-­
fit=no">
    <!-- Bootstrap CSS -->
    <link
      rel="stylesheet"
       href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/
css/bootstrap.min.css">

  </head>
  <body>

<main id="root">Rails backend test message</main>
asd
    <script
      src="https://code.jquery.com/jquery-3.2.1.slim.min.js">
    </script>
    <script
       src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/
1.12.9/umd/popper.min.js">
    </script>
    <script
       src="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/js/
bootstrap.min.js">
    </script>
  </body>
</html>
```

luego agregamos las rutas al `routes.rb`:
```ruby
Rails.application.routes.draw do
  root to: 'messages#root'
  mount ActionCable.server, at: '/cable'
end
```

Aqui ya deberia correr:

## Instalar React y Webpack:
```
npm init
npm install webpack webpack-cli react react-dom @babel/core @babel/preset-react @babel/preset-env babel-loader
```
Si hay errores con la instalacion de los paquetes es posible que halla errores de version:

+  Borrar `package-lock.json`
+  Borrar `node_modules`
+  `npm install` 

creamos en el proyecto lo siguiente:
```
frontend
├── ChatRoom.js
├── index.js
└── MessageForm.js
```


agregamos al `index.js`:
```js
import React from "react";
import ReactDOM from "react-dom";

document.addEventListener("DOMContentLoaded", () => {
  const root = document.getElementById("root");
  ReactDOM.render(<h1>React frontend test message</h1>, root);
});
```

y creamos los componentes en React, en el archivo `ChatRoom.js`:
```jsx
import React from "react";
import MessageForm from "./MessageForm.js";

class ChatRoom extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      messages: []
    };

    this.bottom = React.createRef();
  }

  componentDidMount() {
    App.cable.subscriptions.create(
      { channel: "ChatChannel" },
      {
        received: data => {
          switch (data.type) {
            case "message":
              this.setState({
                messages: this.state.messages.concat(data.message)
              });
              break;
            case "messages":
              this.setState({
                messages: data.messages
              });
              break;
          }
        },
        speak: function(data) {
          return this.perform("speak", data);
        },
        load: function() {
          return this.perform("load");
        }
      }
    );
  }

  loadChat(e) {
    e.preventDefault();
    App.cable.subscriptions.subscriptions[0].load();
  }

  componentDidUpdate() {
    this.bottom.current.scrollIntoView();
  }

  render() {
    const messageList = this.state.messages.map(message => {
      return (
        <li key={message.id}>
          {message}
          <div ref={this.bottom} />
        </li>
      );
    });

    return (
      <div className="chatroom-container">
        <div>ChatRoom</div>
        <button className="load-button" onClick={this.loadChat.bind(this)}>
          Load Chat History
        </button>
        <div className="message-list">{messageList}</div>
        <MessageForm />
      </div>
    );
  }
}

export default ChatRoom;
```
y creamos el componente del mensaje individual:
```jsx
import React from "react";

class MessageForm extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      body: ""
    };
  }

  update(field) {
    return e =>
      this.setState({
        [field]: e.currentTarget.value
      });
  }

  handleSubmit(e) {
    e.preventDefault();
    App.cable.subscriptions.subscriptions[0].speak({
      message: this.state.body
    });
    this.setState({
      body: ""
    });
  }

  render() {
    return (
      <div>
        <form onSubmit={this.handleSubmit.bind(this)}>
          <input
            type="text"
            value={this.state.body}
            onChange={this.update("body")}
            placeholder="Type message here"
          />
          <input type="submit" />
        </form>
      </div>
    );
  }
}

export default MessageForm;
```


y creamos en el proyecto un archivo llamado `webpack.config.js`, este posee los datos de la compilacion de webpack en donde este generara un `bundle.js` que poseera todos los demas js para cargar rapido:
```js
const path = require("path");

module.exports = {
  entry: path.join(__dirname, "frontend", "index.js"),
  output: {
    path: path.join(__dirname, "app", "assets", "javascript"),
    filename: "bundle.js"
  },
  resolve: {
    extensions: [".js", ".jsx", "*"]
  },
  module: {
    rules: [
      {
        exclude: /(node_modules)/,
        use: {
          loader: "babel-loader",
          query: {
            presets: ["@babel/env", "@babel/react"]
          }
        }
      }
    ]
  },
  devtool: "source-map"
};
```

y corremos:
```
yarn  webpack --mode=development
```

### Errores:
Agregamos coffeescript aunque no es indispensable
```
gem 'coffee-rails'
```

Los js de frontend si se quedan asi
las principales diferencias son en los JS, no hay problemas con ruby

En la carpeta `app/javascript/packs` van los archivos que instala la gema de rails, pueden eliminarse libremente

En views sustituir la llamada de los javascripts con :
```
<%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
```

El archivo `app/channels/chat_channel.rb` es modificado:
```ruby
class ChatChannel < ApplicationCable::Channel
  def subscribed
    stream_for 'chat_channel'
  end

  def speak(data)
    message = Message.new(body: data['message'])
    if message.save
      socket = {
        message: message.body,
        type: 'message'
      }
      ChatChannel.broadcast_to('chat_channel', socket)
    end
  end

  def load
    messages = Message.all.collect(&:body)

    socket = {
      messages: messages,
      type: 'messages'
    }

    ChatChannel.broadcast_to('chat_channel', socket)
  end

  def unsubscribed; end
end

```
En `app/assets/javascripts` debe ir un archivo para manejar el cable, `chat.js`, este debe generarlo rails pero ahora ya lo maneja de manera distinta asi que lo creamos individualmente:
```js

// Action Cable provides the framework to deal with WebSockets in Rails.
// You can generate new channels where WebSocket features live using the `rails generate channel` command.
//
//= require action_cable
//= require_self


(function() {
  this.App || (this.App = {});

  App.cable = ActionCable.createConsumer();

}).call(this);

````

Si hay errores en la llamada de carpetas eliminamos esta lineaen el `Chat.js`:
```
//= require_tree ./channels
```
Si dejamos los archivos por default que genera Rails 6 no pasa nada pero sin esa llamada simplemene al archivo chat.js simplemente no corre

El contenido de la carpeta `assets/javascript` no es necesario,quiza solo el index pero no es importante

Se agregan los css al `assets/stylesheets/messagess.scss` y esto le da tooda la presentacion al chat:
```css

* {
  margin: 2px;
  padding: 5px;
}

li {
  list-style: decimal;
}

.chatroom-container {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.message-list {
  overflow-y: scroll;
  height: 300px;
  width: 400px;
  border: solid gray 1px;
}

.load-button {
  position: absolute;
  top: 5px;
  right: 5px;
}

```
tambien instalamos paquetes por si acaso.

```
yarn add --dev webpack-dev-server 
yarn add react-dom react-on-rails
yarn add webpack-cli -D
```

si hay problemas por ejecutar con sudo cambiamos los propietarios con:
```
sudo chown -R oka package.json 
```

action-cable-and-react-9a00be5e391b
