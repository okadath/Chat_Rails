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

y creamos en el proyecto un archivo llamado `webpack.config.js`:
```js
const path = require("path");

module.exports = {
  entry: path.join(__dirname, "frontend", "index.js"),
  output: {
    path: path.join(__dirname, "app", "assets", "javascripts"),
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

## errores:
agregamos cofescript aunque no es indispensable
```
gem 'coffee-rails'
```

los js de frontend si se quedan asi
las principales diferencias son en los JS, no hay problemas con ruby

en la carpeta `app/javascript/packs` van los archivos que usan para react

en viewas sustituir la llamada de los javascripts con :
```
<%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
```

el archivo `app/channels/chat_channel.rb` es modificado:
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
Y no se si estos archivos en la carpeta `app/assets/javascripts` debian ser producidos por rails o si el los produjo, mi maquina produjo unos archivos totalmente diferentes, los copie del repositorio original asi que aun hay que analizarlos

se agregan los css al `assets/stylesheets/messagess.scss` y esto le da tooda la presentacion al chat:
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
yarn add react-dom react-on-rails
yarn add --dev webpack-dev-server 
yarn add react-dom react-on-rails
yarn add webpack-cli -D
```

si hay problemas por ejecutar con sudo cambiamos los propietarios con:
```
sudo chown -R oka package.json 
```