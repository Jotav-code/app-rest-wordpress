# Mapa geral do que eu entendi

## Criando primeira parte da api. Api de usuario

- O que eu preciso para criar essa parte da api?

  ### Simples, eu vou ter uma função que vai ser responsável por fazer um request da minha api em que, ela vai receber um request e vai reornar um response, exemplo abaixo

  ``` php
  
  function api_usuario_post(request){
    // exemplo de como acontece

    $response = array(
      'nome' => 'João'.
      'email' => 'joaovitor646260@gmail.com'
    );

    return rest_ensure_response($response);
  }
  
  ```

   ### Mas ainda não acabou, precisamos registrar essa api no wordpress, vamos ter uma função, que já é padrão do wordpress para registrar api. Vamos colocar a lógica do que ela vai fazer, qual endpoint ela tem que acessar **(methods)** , qual método e qual função ela vai ativar **(callback)** Podemos encontrar fácil essa função na documentação. Essa função vai receber a rota que queremos dar à nossa api, por exemplo... quero acessar a api da seguinte forma jota.com/wp-json/api/usuario

  ``` php

  function registrar_api_usuario(){
    // ela precisa receber uma função do wordpress para registrar, junto passamos o parametro
    register_rest_router('api' , '/usuario', array(
      array(
        'methods' => 'POST' // ou e mais recomendado 'WP_REST_Server::CREATABLE',
        'callback' => 'api_usuario_post'
      )
    ))
    
  }

  // Por ultimo, dizemos quando essa função será ativada

  add_action('rest_api_init' ,'registrar_api_usuario')
  ```
