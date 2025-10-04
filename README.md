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

  ### O que são esses métodos utilizados

  - rest_ensure_response
  
    Serve para garantir que a api vai retonar uma responsta no padrão, O rest_ensure_response faz essa conversão de forma segura → evita bugs e mantém padrão na resposta (status code, headers, JSON bem formatado). sem ela, teriamos que retornar um array, ou seja, é mais prático usar a função do WP

  - register_rest_router o que são os 3 paramatros e pq tenho 2 array aninhado?
    
    1. Primeiro argumento serve para descrever a 'categoria' da minha api, o Segundo argumento é a rota do meu endpoint, já o terceiro é o comportamento da minha api
    2. temos dois array por motivos de... podemos ter mais de um funcionamento nessa api, por exemplo, nessa mesma rota eu posso ter um método get e um post, o mais externo vai segurar os métodos
    3. ``` php
         register_rest_route('api', '/usuario', array(
            array(
                'methods' => 'GET',
                'callback' => 'listar_usuarios',
                  ),
            array(
                'methods' => 'POST',
                'callback' => 'criar_usuario',
                ),
            ));
       ```
    4. WP_REST_Server::CREATABLE. Isso é uma constante do WordPress que significa: esse endpoint aceita métodos de criação de recurso → ou seja, POST. É igual a escrever 'POST', só que mais semântico e compatível.
   
### Criando usuário

- O que a gente vai precisar?
  Lembra do request, que a gente passou como parâmetro da função? Vamos usar ele para separar nossos dados, pegar nome, senha, email... Depois disso temos que fazer uma verificação para saber se o usuário já existe em nosso banco de dados, vamos usar uma função do wordpress para fazer isso **email_exists** vamos fazer isso para o email e para o username, que também será o email **username_exists**.
```php
function api_usuario_post($request) {

  $email = sanitize_email($request['email']);
  $nome = sanitize_text_field($request['nome']);
  $senha = sanitize_text_field($request['senha']);
  $cep = sanitize_text_field($request['cep']);
  $rua = sanitize_text_field($request['rua']);
  $numero = sanitize_text_field($request['numero']);
  $bairro = sanitize_text_field($request['bairro']);
  $cidade = sanitize_text_field($request['cidade']);
  $estado = sanitize_text_field($request['estado']);
```
  Depois de verificar que o usuário não existe, vamos adicionar ele em nosso banco de dados do wordpress, utilizando a função **wp_create_user**, passamos como argumento username, senha e email, mas em nossa api o username será o email. Muito importante lembrar que o **wp_create_user** nos retorna um ID único do usuário, ele vai ser importante pra gente.
  ```php
  $email_exist = email_exists($email);
  $username_exist = username_exists($email);

  if(!$email_exist && !$username_exist && $senha && $nome && $cep && $rua && $numero && $bairro && $cidade && $estado) {

    $user_id = wp_create_user($email, $senha, $email);
  ```

  Depois de criar o usuário, precisamos fazer um leve update nele, colocando seu ID, fisrt name, role(Sua função) e display_name. Além disso, precisamos criar os meta dados restantes no banco de dados(cidade, cep, estado...)
  ```php
    $update_user_data = array(
      'ID' => $user_id,
      'display_name' => $nome,
      'first_name' => $nome,
      'role' => 'subscriber'
    );
    wp_update_user($update_user_data);

    update_user_meta( $user_id,'cep', $cep);
    update_user_meta( $user_id,'rua', $rua);
    update_user_meta( $user_id,'numero', $numero);
    update_user_meta( $user_id,'bairro', $bairro);
    update_user_meta( $user_id,'cidade', $cidade);
    update_user_meta( $user_id,'estado', $estado);

  } else {
    $response = new WP_Error('email', 'Email já cadastrado', array('status' => 403));
  }
  return rest_ensure_response($response);
}
  ```
**Código Final**
```php
  <?php

function api_usuario_post($request) {

  $email = sanitize_email($request['email']);
  $nome = sanitize_text_field($request['nome']);
  $senha = sanitize_text_field($request['senha']);
  $cep = sanitize_text_field($request['cep']);
  $rua = sanitize_text_field($request['rua']);
  $numero = sanitize_text_field($request['numero']);
  $bairro = sanitize_text_field($request['bairro']);
  $cidade = sanitize_text_field($request['cidade']);
  $estado = sanitize_text_field($request['estado']);

  $email_exist = email_exists($email);
  $username_exist = username_exists($email);

  if(!$email_exist && !$username_exist && $senha && $nome && $cep && $rua && $numero && $bairro && $cidade && $estado) {

    $user_id = wp_create_user($email, $senha, $email);

    $update_user_data = array(
      'ID' => $user_id,
      'display_name' => $nome,
      'first_name' => $nome,
      'role' => 'subscriber'
    );
    wp_update_user($update_user_data);

    update_user_meta( $user_id,'cep', $cep);
    update_user_meta( $user_id,'rua', $rua);
    update_user_meta( $user_id,'numero', $numero);
    update_user_meta( $user_id,'bairro', $bairro);
    update_user_meta( $user_id,'cidade', $cidade);
    update_user_meta( $user_id,'estado', $estado);

  } else {
    $response = new WP_Error('email', 'Email já cadastrado', array('status' => 403));
  }
  return rest_ensure_response($response);
}

function registrar_api_usuario_post() {
  register_rest_route('api', '/usuario', array(
    array(
      'methods' => WP_REST_Server::CREATABLE,
      'callback' => 'api_usuario_post',
    ),
  ));
}

add_action('rest_api_init', 'registrar_api_usuario_post');


?>
```
