# Test Chat Rails
Escencialmente Django es una mierda por que no facilita las API REST pero era dificil hacer RealTime Apps en Rails hasta que crearon ActionCable, si esto funciona sera un buen backend para React:

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

Instalar yarn y actualizar:
```
rails webpacker:install
```


ya con uno creado si no esta activo, (no se si va con el create):
```
/bin/bash --login
rvm use ruby@rails --create
```

## Proyecto

crear:
```
rails new actioncable_app
cd actioncable_app
rails db:create
```
y creamos los modelos, controladores y el Channel:
```shell
rails g model Message body:string
rails g controller Messages
rails g channel Chat
rails db:migrate
```
si usamos coffescript borrar el `chat.coffee` por que usaremos rect

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


'aqui ya deberia correr:

## Instalar React y Webpack:
```
npm init
npm install webpack webpack-cli react react-dom @babel/core @babel/preset-react @babel/preset-env babel-loader
```
Si hay errores con la instalacion de los paquetes es posible que halla errores de version:

+  Borrar `package-lock.json`
+  Borrar `node_modules`
+  `npm install` 
