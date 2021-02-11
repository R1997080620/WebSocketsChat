Dans cet article, nous allons créer une application de chat PHP et Mysql simple en utilisant PHP Web Socket Ratchet. Si vous recherchez Comment créer une application de chat en direct ou en temps réel en PHP, vous êtes au bon endroit, car dans cet article, nous vous avons expliqué comment créer une application de chat en direct PHP avec WebSockets. Dans cet article, nous allons créer une application de chat dans laquelle plusieurs utilisateurs peuvent discuter avec d'autres utilisateurs.
- - -
- - -

Ceci est un article très intéressant sur Comment créer une application de chat en PHP sans utiliser Ajax. Dans l'un de nos articles précédents, nous avons déjà publié un tutoriel sur l' application Chat en PHP utilisant ajax . Mais comme nous le savons dans le système de chat, l'application de chat basée sur Ajax en PHP n'est pas la meilleure solution, car Ajax augmentera la charge sur le serveur. Donc, pour cela, nous avons utilisé Ratchet PHP WebSockets pour créer une application de chat en direct en temps réel en PHP et dans cette application de chat, nous stockerons les données de chat dans la base de données Mysql lorsqu'un utilisateur a posté une discussion sous ce système.
***
***

### Qu'est-ce que WebSockets?

WebSockets est un système bidirectionnel et duplex intégral qui fournit une connexion permanente d'un navigateur Web à notre serveur. Ainsi, lorsque la connexion WebSocket a été établie dans notre navigateur, la connexion sera ouverte jusqu'à ce que le client ou le serveur décide de fermer cette connexion. Ainsi, avec cette connexion ouverte, l'utilisateur ou notre serveur peut envoyer ou recevoir des données de chat à tout moment l'un à l'autre et cela rendra notre programmation Web complètement basée sur les événements et pas seulement sur le démarrage de l'utilisateur. Les autres avantages de ces Websockets sont que, en même temps sur un seul serveur en cours d'exécution, il connectera toutes les connexions et nous permettra de communiquer avec n'importe quel nombre de connexions en direct à tout moment. Ce sont donc tous les avantages de ces WebSockets dans cette programmation Web.

Sur la base de ces avantages, nous avons utilisé des WebSockets PHP comme Ratchet pour créer une application de chat en PHP et Mysql. Dans cet article, nous allons vous apprendre étape par étape comment créer une application de chat en PHP en utilisant WebSocket à partir de zéro. Dans ce didacticiel de développement Web PHP, vous pouvez apprendre comment créer rapidement une application de chat avec RatChet à l'aide d'un script PHP et d'une base de données Mysql.

### Base de données de l'application de chat

Avant d'apprendre l'application de chat, vous devez d'abord créer des tables pour stocker les données de l'application de chat dans la base de données Mysql. Vous devez donc d'abord créer des tables en exécutant le script Sql suivant, vous pouvez créer des tables d'application de chat dans votre base de données mysql.

![WebSocketsChat](https://1.bp.blogspot.com/-xpt3wdo3qmg/X_gFH47rKwI/AAAAAAAABlI/NG5M0Pwz2IE4TlhaQ9y_LFty_njIsXxQACLcBGAsYHQ/s640/chat-application-in-php-using-websockets-blog.jpg)

### Base de données de l'application de chat

Avant d'apprendre l'application de chat, vous devez d'abord créer des tables pour stocker les données de l'application de chat dans la base de données Mysql. Vous devez donc d'abord créer des tables en exécutant le script Sql suivant, vous pouvez créer des tables d'application de chat dans votre base de données mysql.

```--
-- Database: `chat`
--

-- --------------------------------------------------------

--
-- Table structure for table `chatrooms`
--

CREATE TABLE `chatrooms` (
  `id` int(11) NOT NULL,
  `userid` int(11) NOT NULL,
  `msg` varchar(200) NOT NULL,
  `created_on` datetime NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

-- --------------------------------------------------------

--
-- Table structure for table `chat_user_table`
--

CREATE TABLE `chat_user_table` (
  `user_id` int(11) NOT NULL,
  `user_name` varchar(250) NOT NULL,
  `user_email` varchar(250) NOT NULL,
  `user_password` varchar(100) NOT NULL,
  `user_profile` varchar(100) NOT NULL,
  `user_status` enum('Disabled','Enable') NOT NULL,
  `user_created_on` datetime NOT NULL,
  `user_verification_code` varchar(100) NOT NULL,
  `user_login_status` enum('Logout','Login') NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

--
-- Indexes for dumped tables
--

--
-- Indexes for table `chatrooms`
--
ALTER TABLE `chatrooms`
  ADD PRIMARY KEY (`id`);

--
-- Indexes for table `chat_user_table`
--
ALTER TABLE `chat_user_table`
  ADD PRIMARY KEY (`user_id`);

--
-- AUTO_INCREMENT for dumped tables
--

--
-- AUTO_INCREMENT for table `chatrooms`
--
ALTER TABLE `chatrooms`
  MODIFY `id` int(11) NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=6;

--
-- AUTO_INCREMENT for table `chat_user_table`
--
ALTER TABLE `chat_user_table`
  MODIFY `user_id` int(11) NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=11;

```

### Code source de l'application de chat PHP

Vous trouverez ci-dessous le fichier de code source de l'application PHP Chat.

**_database/Database_connection.php_**

Ce fichier que nous utiliserons pour établir une connexion à la base de données sous cette application de chat.

```
<?php

//Database_connection.php

class Database_connection
{
	function connect()
	{
		$connect = new PDO("mysql:host=localhost; dbname=chat", "root", "");

		return $connect;
	}
}

?>
```

**_database/ChatUser.php_**

Ce fichier de classe **ChatUser.php** que nous utilisons pour traiter les données utilisateur côté serveur. Sous ce fichier de classe, nous avons une méthode make set et get pour les données utilisateur et après cela, nous avons une méthode make pour vérifier que l'utilisateur s'inscrit déjà ou non et enfin nous devons insérer les données utilisateur dans la table mysql.

```
<?php

//ChatUser.php

class ChatUser
{
	private $user_id;
	private $user_name;
	private $user_email;
	private $user_password;
	private $user_profile;
	private $user_status;
	private $user_created_on;
	private $user_verification_code;
	private $user_login_status;
	public $connect;

	public function __construct()
	{
		require_once('Database_connection.php');

		$database_object = new Database_connection;

		$this->connect = $database_object->connect();
	}

	function setUserId($user_id)
	{
		$this->user_id = $user_id;
	}

	function getUserId()
	{
		return $this->user_id;
	}

	function setUserName($user_name)
	{
		$this->user_name = $user_name;
	}

	function getUserName()
	{
		return $this->user_name;
	}

	function setUserEmail($user_email)
	{
		$this->user_email = $user_email;
	}

	function getUserEmail()
	{
		return $this->user_email;
	}

	function setUserPassword($user_password)
	{
		$this->user_password = $user_password;
	}

	function getUserPassword()
	{
		return $this->user_password;
	}

	function setUserProfile($user_profile)
	{
		$this->user_profile = $user_profile;
	}

	function getUserProfile()
	{
		return $this->user_profile;
	}

	function setUserStatus($user_status)
	{
		$this->user_status = $user_status;
	}

	function getUserStatus()
	{
		return $this->user_status;
	}

	function setUserCreatedOn($user_created_on)
	{
		$this->user_created_on = $user_created_on;
	}

	function getUserCreatedOn()
	{
		return $this->user_created_on;
	}

	function setUserVerificationCode($user_verification_code)
	{
		$this->user_verification_code = $user_verification_code;
	}

	function getUserVerificationCode()
	{
		return $this->user_verification_code;
	}

	function setUserLoginStatus($user_login_status)
	{
		$this->user_login_status = $user_login_status;
	}

	function getUserLoginStatus()
	{
		return $this->user_login_status;
	}

	function make_avatar($character)
	{
	    $path = "images/". time() . ".png";
		$image = imagecreate(200, 200);
		$red = rand(0, 255);
		$green = rand(0, 255);
		$blue = rand(0, 255);
	    imagecolorallocate($image, $red, $green, $blue);  
	    $textcolor = imagecolorallocate($image, 255,255,255);

	    $font = dirname(__FILE__) . '/font/arial.ttf';

	    imagettftext($image, 100, 0, 55, 150, $textcolor, $font, $character);
	    imagepng($image, $path);
	    imagedestroy($image);
	    return $path;
	}

	function get_user_data_by_email()
	{
		$query = "
		SELECT * FROM chat_user_table 
		WHERE user_email = :user_email
		";

		$statement = $this->connect->prepare($query);

		$statement->bindParam(':user_email', $this->user_email);

		if($statement->execute())
		{
			$user_data = $statement->fetch(PDO::FETCH_ASSOC);
		}
		return $user_data;
	}

	function save_data()
	{
		$query = "
		INSERT INTO chat_user_table (user_name, user_email, user_password, user_profile, user_status, user_created_on, user_verification_code) 
		VALUES (:user_name, :user_email, :user_password, :user_profile, :user_status, :user_created_on, :user_verification_code)
		";
		$statement = $this->connect->prepare($query);

		$statement->bindParam(':user_name', $this->user_name);

		$statement->bindParam(':user_email', $this->user_email);

		$statement->bindParam(':user_password', $this->user_password);

		$statement->bindParam(':user_profile', $this->user_profile);

		$statement->bindParam(':user_status', $this->user_status);

		$statement->bindParam(':user_created_on', $this->user_created_on);

		$statement->bindParam(':user_verification_code', $this->user_verification_code);

		if($statement->execute())
		{
			return true;
		}
		else
		{
			return false;
		}
	}

	function is_valid_email_verification_code()
	{
		$query = "
		SELECT * FROM chat_user_table 
		WHERE user_verification_code = :user_verification_code
		";

		$statement = $this->connect->prepare($query);

		$statement->bindParam(':user_verification_code', $this->user_verification_code);

		$statement->execute();

		if($statement->rowCount() > 0)
		{
			return true;
		}
		else
		{
			return false;
		}
	}

	function enable_user_account()
	{
		$query = "
		UPDATE chat_user_table 
		SET user_status = :user_status 
		WHERE user_verification_code = :user_verification_code
		";

		$statement = $this->connect->prepare($query);

		$statement->bindParam(':user_status', $this->user_status);

		$statement->bindParam(':user_verification_code', $this->user_verification_code);

		if($statement->execute())
		{
			return true;
		}
		else
		{
			return false;
		}
	}

	function update_user_login_data()
	{
		$query = "
		UPDATE chat_user_table 
		SET user_login_status = :user_login_status 
		WHERE user_id = :user_id
		";

		$statement = $this->connect->prepare($query);

		$statement->bindParam(':user_login_status', $this->user_login_status);

		$statement->bindParam(':user_id', $this->user_id);

		if($statement->execute())
		{
			return true;
		}
		else
		{
			return false;
		}
	}

	function get_user_data_by_id()
	{
		$query = "
		SELECT * FROM chat_user_table 
		WHERE user_id = :user_id";

		$statement = $this->connect->prepare($query);

		$statement->bindParam(':user_id', $this->user_id);

		try
		{
			if($statement->execute())
			{
				$user_data = $statement->fetch(PDO::FETCH_ASSOC);
			}
			else
			{
				$user_data = array();
			}
		}
		catch (Exception $error)
		{
			echo $error->getMessage();
		}
		return $user_data;
	}

	function upload_image($user_profile)
	{
		$extension = explode('.', $user_profile['name']);
		$new_name = rand() . '.' . $extension[1];
		$destination = 'images/' . $new_name;
		move_uploaded_file($user_profile['tmp_name'], $destination);
		return $destination;
	}

	function update_data()
	{
		$query = "
		UPDATE chat_user_table 
		SET user_name = :user_name, 
		user_email = :user_email, 
		user_password = :user_password, 
		user_profile = :user_profile  
		WHERE user_id = :user_id
		";

		$statement = $this->connect->prepare($query);

		$statement->bindParam(':user_name', $this->user_name);

		$statement->bindParam(':user_email', $this->user_email);

		$statement->bindParam(':user_password', $this->user_password);

		$statement->bindParam(':user_profile', $this->user_profile);

		$statement->bindParam(':user_id', $this->user_id);

		if($statement->execute())
		{
			return true;
		}
		else
		{
			return false;
		}
	}

	function get_user_all_data()
	{
		$query = "
		SELECT * FROM chat_user_table 
		";

		$statement = $this->connect->prepare($query);

		$statement->execute();

		$data = $statement->fetchAll(PDO::FETCH_ASSOC);

		return $data;
	}

}

?>
```

**_database/ChatRooms.php**

Cette classe que nous utiliserons pour le fonctionnement de la base de données des données des messages de discussion. Dans cette classe, nous allons créer et obtenir une fonction pour l'opération liée à la base de données de données de discussion de processus. En utilisant cette classe, nous insérerons ou stockerons un message de discussion dans la base de données Mysql et récupérerons les données de discussion de la base de données Mysql pour les afficher sur la page Web.

~~~
<?php 
	
class ChatRooms
{
	private $chat_id;
	private $user_id;
	private $message;
	private $created_on;
	protected $connect;

	public function setChatId($chat_id)
	{
		$this->chat_id = $chat_id;
	}

	function getChatId()
	{
		return $this->chat_id;
	}

	function setUserId($user_id)
	{
		$this->user_id = $user_id;
	}

	function getUserId()
	{
		return $this->user_id;
	}

	function setMessage($message)
	{
		$this->message = $message;
	}

	function getMessage()
	{
		return $this->message;
	}

	function setCreatedOn($created_on)
	{
		$this->created_on = $created_on;
	}

	function getCreatedOn()
	{
		return $this->created_on;
	}

	public function __construct()
	{
		require_once("Database_connection.php");

		$database_object = new Database_connection;

		$this->connect = $database_object->connect();
	}

	function save_chat()
	{
		$query = "
		INSERT INTO chatrooms 
			(userid, msg, created_on) 
			VALUES (:userid, :msg, :created_on)
		";

		$statement = $this->connect->prepare($query);

		$statement->bindParam(':userid', $this->user_id);

		$statement->bindParam(':msg', $this->message);

		$statement->bindParam(':created_on', $this->created_on);

		$statement->execute();
	}

	function get_all_chat_data()
	{
		$query = "
		SELECT * FROM chatrooms 
			INNER JOIN chat_user_table 
			ON chat_user_table.user_id = chatrooms.userid 
			ORDER BY chatrooms.id ASC
		";

		$statement = $this->connect->prepare($query);

		$statement->execute();

		return $statement->fetchAll(PDO::FETCH_ASSOC);
	}
}
	
?>
~~~

**_register.php_**

Ce fichier que nous utilisons pour l'enregistrement des utilisateurs de l'application Chat. L'utilisateur peut s'inscrire à partir de cette page Web

```
<?php

use PHPMailer\PHPMailer\PHPMailer;
use PHPMailer\PHPMailer\SMTP;
use PHPMailer\PHPMailer\Exception;

require 'vendor/autoload.php';

$error = '';

$success_message = '';

if(isset($_POST["register"]))
{
    session_start();

    if(isset($_SESSION['user_data']))
    {
        header('location:chatroom.php');
    }

    require_once('database/ChatUser.php');

    $user_object = new ChatUser;

    $user_object->setUserName($_POST['user_name']);

    $user_object->setUserEmail($_POST['user_email']);

    $user_object->setUserPassword($_POST['user_password']);

    $user_object->setUserProfile($user_object->make_avatar(strtoupper($_POST['user_name'][0])));

    $user_object->setUserStatus('Disabled');

    $user_object->setUserCreatedOn(date('Y-m-d H:i:s'));

    $user_object->setUserVerificationCode(md5(uniqid()));

    $user_data = $user_object->get_user_data_by_email();

    if(is_array($user_data) && count($user_data) > 0)
    {
        $error = 'This Email Already Register';
    }
    else
    {
        if($user_object->save_data())
        {

            $mail = new PHPMailer(true);

            $mail->isSMTP();

            $mail->Host = 'smtpout.secureserver.net';

            $mail->SMTPAuth = true;

            $mail->Username   = 'xxxxx';                     // SMTP username
            $mail->Password   = 'xxxxxx';

            $mail->SMTPSecure = PHPMailer::ENCRYPTION_STARTTLS;

            $mail->Port = 80;

            $mail->setFrom('tutorial@webslesson.info', 'Webslesson');

            $mail->addAddress($user_object->getUserEmail());

            $mail->isHTML(true);

            $mail->Subject = 'Registration Verification for Chat Application Demo';

            $mail->Body = '
            <p>Thank you for registering for Chat Application Demo.</p>
                <p>This is a verification email, please click the link to verify your email address.</p>
                <p><a href="http://localhost/tutorial/chat_application/verify.php?code='.$user_object->getUserVerificationCode().'">Click to Verify</a></p>
                <p>Thank you...</p>
            ';

            $mail->send();


            $success_message = 'Verification Email sent to ' . $user_object->getUserEmail() . ', so before login first verify your email';
        }
        else
        {
            $error = 'Something went wrong try again';
        }
    }

}


?>

<!DOCTYPE html>
<html lang="en">

<head>

    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <meta name="description" content="">
    <meta name="author" content="">

    <title>Register | PHP Chat Application using Websocket</title>

    <!-- Bootstrap core CSS -->
    <link href="vendor-front/bootstrap/bootstrap.min.css" rel="stylesheet">

    <link href="vendor-front/fontawesome-free/css/all.min.css" rel="stylesheet" type="text/css">

    <link rel="stylesheet" type="text/css" href="vendor-front/parsley/parsley.css"/>

    <!-- Bootstrap core JavaScript -->
    <script src="vendor-front/jquery/jquery.min.js"></script>
    <script src="vendor-front/bootstrap/js/bootstrap.bundle.min.js"></script>

    <!-- Core plugin JavaScript-->
    <script src="vendor-front/jquery-easing/jquery.easing.min.js"></script>

    <script type="text/javascript" src="vendor-front/parsley/dist/parsley.min.js"></script>
</head>

<body>

    <div class="containter">
        <br />
        <br />
        <h1 class="text-center">Chat Application in PHP & MySql using WebSocket - Email Verification</h1>
        
        <div class="row justify-content-md-center">
            <div class="col col-md-4 mt-5">
                <?php
                if($error != '')
                {
                    echo '
                    <div class="alert alert-warning alert-dismissible fade show" role="alert">
                      '.$error.'
                      <button type="button" class="close" data-dismiss="alert" aria-label="Close">
                        <span aria-hidden="true">&times;</span>
                      </button>
                    </div>
                    ';
                }

                if($success_message != '')
                {
                    echo '
                    <div class="alert alert-success">
                    '.$success_message.'
                    </div>
                    ';
                }
                ?>
                <div class="card">
                    <div class="card-header">Register</div>
                    <div class="card-body">

                        <form method="post" id="register_form">

                            <div class="form-group">
                                <label>Enter Your Name</label>
                                <input type="text" name="user_name" id="user_name" class="form-control" data-parsley-pattern="/^[a-zA-Z\s]+$/" required />
                            </div>

                            <div class="form-group">
                                <label>Enter Your Email</label>
                                <input type="text" name="user_email" id="user_email" class="form-control" data-parsley-type="email" required />
                            </div>

                            <div class="form-group">
                                <label>Enter Your Password</label>
                                <input type="password" name="user_password" id="user_password" class="form-control" data-parsley-minlength="6" data-parsley-maxlength="12" data-parsley-pattern="^[a-zA-Z]+$" required />
                            </div>

                            <div class="form-group text-center">
                                <input type="submit" name="register" class="btn btn-success" value="Register" />
                            </div>

                        </form>
                        
                    </div>
                </div>
                
            </div>
        </div>
    </div>

</body>

</html>

<script>

$(document).ready(function(){

    $('#register_form').parsley();

});

</script>
```

**_index.php_**

Sous ce fichier, nous créerons une page de connexion pour cette application de chat.

```
<?php

session_start();

$error = '';

if(isset($_SESSION['user_data']))
{
    header('location:chatroom.php');
}

if(isset($_POST['login']))
{
    require_once('database/ChatUser.php');

    $user_object = new ChatUser;

    $user_object->setUserEmail($_POST['user_email']);

    $user_data = $user_object->get_user_data_by_email();

    if(is_array($user_data) && count($user_data) > 0)
    {
        if($user_data['user_status'] == 'Enable')
        {
            if($user_data['user_password'] == $_POST['user_password'])
            {
                $user_object->setUserId($user_data['user_id']);
                $user_object->setUserLoginStatus('Login');

                if($user_object->update_user_login_data())
                {
                    $_SESSION['user_data'][$user_data['user_id']] = [
                        'id'    =>  $user_data['user_id'],
                        'name'  =>  $user_data['user_name'],
                        'profile'   =>  $user_data['user_profile']
                    ];

                    header('location:chatroom.php');

                }
            }
            else
            {
                $error = 'Wrong Password';
            }
        }
        else
        {
            $error = 'Please Verify Your Email Address';
        }
    }
    else
    {
        $error = 'Wrong Email Address';
    }
}

?>

<!DOCTYPE html>
<html lang="en">

<head>

    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <meta name="description" content="">
    <meta name="author" content="">

    <title>Login | PHP Chat Application using Websocket</title>

    <!-- Bootstrap core CSS -->
    <link href="vendor-front/bootstrap/bootstrap.min.css" rel="stylesheet">

    <link href="vendor-front/fontawesome-free/css/all.min.css" rel="stylesheet" type="text/css">

    <link rel="stylesheet" type="text/css" href="vendor-front/parsley/parsley.css"/>

    <!-- Bootstrap core JavaScript -->
    <script src="vendor-front/jquery/jquery.min.js"></script>
    <script src="vendor-front/bootstrap/js/bootstrap.bundle.min.js"></script>

    <!-- Core plugin JavaScript-->
    <script src="vendor-front/jquery-easing/jquery.easing.min.js"></script>

    <script type="text/javascript" src="vendor-front/parsley/dist/parsley.min.js"></script>
</head>

<body>

    <div class="containter">
        <br />
        <br />
        <h1 class="text-center">Chat Application in PHP & MySql using WebSocket - Login</h1>
        <div class="row justify-content-md-center mt-5">
            
            <div class="col-md-4">
               <?php
               if(isset($_SESSION['success_message']))
               {
                    echo '
                    <div class="alert alert-success">
                    '.$_SESSION["success_message"] .'
                    </div>
                    ';
                    unset($_SESSION['success_message']);
               }

               if($error != '')
               {
                    echo '
                    <div class="alert alert-danger">
                    '.$error.'
                    </div>
                    ';
               }
               ?>
                <div class="card">
                    <div class="card-header">Login</div>
                    <div class="card-body">
                        <form method="post" id="login_form">
                            <div class="form-group">
                                <label>Enter Your Email Address</label>
                                <input type="text" name="user_email" id="user_email"  class="form-control" data-parsley-type="email" required />
                            </div>
                            <div class="form-group">
                                <label>Enter Your Password</label>
                                <input type="password" name="user_password" id="user_password" class="form-control" required />
                            </div>
                            <div class="form-group text-center">
                                <input type="submit" name="login" id="login" class="btn btn-primary" value="Login" />
                            </div>
                        </form>
                    </div>  
                </div>
            </div>
        </div>
    </div>

</body>

</html>

<script>

$(document).ready(function(){
    
    $('#login_form').parsley();
    
});

</script>
```

**_verify.php_**

Ce code source de fichier vérifiera l'adresse e-mail de l'utilisateur et activera le compte utilisateur pour la connexion au système de chat.

```
<?php

//verify.php

$error = '';

session_start();

if(isset($_GET['code']))
{
    require_once('database/ChatUser.php');

    $user_object = new ChatUser;

    $user_object->setUserVerificationCode($_GET['code']);

    if($user_object->is_valid_email_verification_code())
    {
        $user_object->setUserStatus('Enable');

        if($user_object->enable_user_account())
        {
            $_SESSION['success_message'] = 'Your Email Successfully verify, now you can login into this chat Application';

            header('location:index.php');
        }
        else
        {
            $error = 'Something went wrong try again....';
        }
    }
    else
    {
        $error = 'Something went wrong try again....';
    }
}

?>

<!DOCTYPE html>
<html lang="en">

<head>

    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <meta name="description" content="">
    <meta name="author" content="">

    <title>Email Verify | PHP Chat Application using Websocket</title>

    <!-- Bootstrap core CSS -->
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css" integrity="sha384-ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T" crossorigin="anonymous">

    <!-- Bootstrap core JavaScript -->
    <script src="https://code.jquery.com/jquery-3.5.1.js" integrity="sha256-QWo7LDvxbWT2tbbQ97B53yJnYU3WhH/C8ycbRAkjPDc=" crossorigin="anonymous"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.7/umd/popper.min.js" integrity="sha384-UO2eT0CpHqdSJQ6hJty5KVphtPhzWj9WO1clHTMGa3JDZwrnQq4sF86dIHNDz0W1" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js" integrity="sha384-JjSmVgyd0p3pXB1rRibZUAYoIIy6OrQ6VrjIEaFf/nJGzIxFDsf4x0xIM+B07jRM" crossorigin="anonymous"></script>

</head>

<body>

    <div class="containter">
        <br />
        <br />
        <h1 class="text-center">PHP Chat Application using Websocket</h1>
        
        <div class="row justify-content-md-center">
            <div class="col col-md-4 mt-5">
            	<div class="alert alert-danger">
            		<h2><?php echo $error; ?></h2>
            	</div>
            </div>
        </div>
    </div>
</body>

</html>
```
**_chatroom.php_**

Ce **chatroom.php** ne sera accessible que par l'utilisateur de connexion, donc une fois que l'utilisateur s'est connecté à cette application de chat, le système redirigera la page vers ce fichier chatroom.php et sous ce fichier, nous créerons une salle de chat, afin que chaque utilisateur puisse discuter avec L'une et l'autre. Sous cette page, l'utilisateur peut voir sa photo de profil avec le lien de modification du profil et du bouton de déconnexion.

```
<?php 
session_start();

if(!isset($_SESSION['user_data']))
{
	header('location:index.php');
}

require('database/ChatUser.php');

require('database/ChatRooms.php');

$chat_object = new ChatRooms;

$chat_data = $chat_object->get_all_chat_data();

$user_object = new ChatUser;

$user_data = $user_object->get_user_all_data();

?>

<!DOCTYPE html>
<html>
<head>
	<title>Chat application in php using web scocket programming</title>
	<!-- Bootstrap core CSS -->
    <link href="vendor-front/bootstrap/bootstrap.min.css" rel="stylesheet">

    <link href="vendor-front/fontawesome-free/css/all.min.css" rel="stylesheet" type="text/css">

    <link rel="stylesheet" type="text/css" href="vendor-front/parsley/parsley.css"/>

    <!-- Bootstrap core JavaScript -->
    <script src="vendor-front/jquery/jquery.min.js"></script>
    <script src="vendor-front/bootstrap/js/bootstrap.bundle.min.js"></script>

    <!-- Core plugin JavaScript-->
    <script src="vendor-front/jquery-easing/jquery.easing.min.js"></script>

    <script type="text/javascript" src="vendor-front/parsley/dist/parsley.min.js"></script>
	<style type="text/css">
		html,
		body {
		  height: 100%;
		  width: 100%;
		  margin: 0;
		}
		#wrapper
		{
			display: flex;
		  	flex-flow: column;
		  	height: 100%;
		}
		#remaining
		{
			flex-grow : 1;
		}
		#messages {
			height: 200px;
			background: whitesmoke;
			overflow: auto;
		}
		#chat-room-frm {
			margin-top: 10px;
		}
		#user_list
		{
			height:450px;
			overflow-y: auto;
		}

		#messages_area
		{
			height: 650px;
			overflow-y: auto;
			background-color:#e6e6e6;
		}

	</style>
</head>
<body>
	<div class="container">
		<br />
        <h3 class="text-center">PHP Chat Application using Websocket - Display User with Online or Offline Status</h3>
        <br />
		<div class="row">
			
			<div class="col-lg-8">
				<div class="card">
					<div class="card-header"><h3>Chat Room</h3></div>
					<div class="card-body" id="messages_area">
					<?php
					foreach($chat_data as $chat)
					{
						if(isset($_SESSION['user_data'][$chat['userid']]))
						{
							$from = 'Me';
							$row_class = 'row justify-content-start';
							$background_class = 'text-dark alert-light';
						}
						else
						{
							$from = $chat['user_name'];
							$row_class = 'row justify-content-end';
							$background_class = 'alert-success';
						}

						echo '
						<div class="'.$row_class.'">
							<div class="col-sm-10">
								<div class="shadow-sm alert '.$background_class.'">
									<b>'.$from.' - </b>'.$chat["msg"].'
									<br />
									<div class="text-right">
										<small><i>'.$chat["created_on"].'</i></small>
									</div>
								</div>
							</div>
						</div>
						';
					}
					?>
					</div>
				</div>

				<form method="post" id="chat_form" data-parsley-errors-container="#validation_error">
					<div class="input-group mb-3">
						<textarea class="form-control" id="chat_message" name="chat_message" placeholder="Type Message Here" data-parsley-maxlength="1000" data-parsley-pattern="/^[a-zA-Z0-9\s]+$/" required></textarea>
						<div class="input-group-append">
							<button type="submit" name="send" id="send" class="btn btn-primary"><i class="fa fa-paper-plane"></i></button>
						</div>
					</div>
					<div id="validation_error"></div>
				</form>
			</div>
			<div class="col-lg-4">
				<?php

				$login_user_id = '';

				foreach($_SESSION['user_data'] as $key => $value)
				{
					$login_user_id = $value['id'];
				?>
				<input type="hidden" name="login_user_id" id="login_user_id" value="<?php echo $login_user_id; ?>" />
				<div class="mt-3 mb-3 text-center">
					<img src="<?php echo $value['profile']; ?>" width="150" class="img-fluid rounded-circle img-thumbnail" />
					<h3 class="mt-2"><?php echo $value['name']; ?></h3>
					<a href="profile.php" class="btn btn-secondary mt-2 mb-2">Edit</a>
					<input type="button" class="btn btn-primary mt-2 mb-2" name="logout" id="logout" value="Logout" />
				</div>
				<?php
				}
				?>

				<div class="card mt-3">
					<div class="card-header">User List</div>
					<div class="card-body" id="user_list">
						<div class="list-group list-group-flush">
						<?php
						if(count($user_data) > 0)
						{
							foreach($user_data as $key => $user)
							{
								$icon = '<i class="fa fa-circle text-danger"></i>';

								if($user['user_login_status'] == 'Login')
								{
									$icon = '<i class="fa fa-circle text-success"></i>';
								}

								if($user['user_id'] != $login_user_id)
								{
									echo '
									<a class="list-group-item list-group-item-action">
										<img src="'.$user["user_profile"].'" class="img-fluid rounded-circle img-thumbnail" width="50" />
										<span class="ml-1"><strong>'.$user["user_name"].'</strong></span>
										<span class="mt-2 float-right">'.$icon.'</span>
									</a>
									';
								}

							}
						}
						?>
						</div>
					</div>
				</div>

			</div>
		</div>
	</div>
</body>
<script type="text/javascript">
	
	$(document).ready(function(){

		var conn = new WebSocket('ws://localhost:8080');
		conn.onopen = function(e) {
		    console.log("Connection established!");
		};

		conn.onmessage = function(e) {
		    console.log(e.data);

		    var data = JSON.parse(e.data);

		    var row_class = '';

		    var background_class = '';

		    if(data.from == 'Me')
		    {
		    	row_class = 'row justify-content-start';
		    	background_class = 'text-dark alert-light';
		    }
		    else
		    {
		    	row_class = 'row justify-content-end';
		    	background_class = 'alert-success';
		    }

		    var html_data = "<div class='"+row_class+"'><div class='col-sm-10'><div class='shadow-sm alert "+background_class+"'><b>"+data.from+" - </b>"+data.msg+"<br /><div class='text-right'><small><i>"+data.dt+"</i></small></div></div></div></div>";

		    $('#messages_area').append(html_data);

		    $("#chat_message").val("");
		};

		$('#chat_form').parsley();

		$('#messages_area').scrollTop($('#messages_area')[0].scrollHeight);

		$('#chat_form').on('submit', function(event){

			event.preventDefault();

			if($('#chat_form').parsley().isValid())
			{

				var user_id = $('#login_user_id').val();

				var message = $('#chat_message').val();

				var data = {
					userId : user_id,
					msg : message
				};

				conn.send(JSON.stringify(data));

				$('#messages_area').scrollTop($('#messages_area')[0].scrollHeight);

			}

		});
		
		$('#logout').click(function(){

			user_id = $('#login_user_id').val();

			$.ajax({
				url:"action.php",
				method:"POST",
				data:{user_id:user_id, action:'leave'},
				success:function(data)
				{
					var response = JSON.parse(data);

					if(response.status == 1)
					{
						conn.close();
						location = 'index.php';
					}
				}
			})

		});

	});
	
</script>
</html>
```

**_action.php_**

Ce fichier a reçu une demande ajax pour la déconnexion de l'utilisateur de l'application utilisateur de chat. Sous ce fichier, nous devons d'abord supprimer la valeur de la variable de session, puis après avoir supprimé toutes les variables de session et donner une valeur de statut à la requête ajax au format json.

```
<?php

//action.php

session_start();

if(isset($_POST['action']) && $_POST['action'] == 'leave')
{
	require('database/ChatUser.php');

	$user_object = new ChatUser;

	$user_object->setUserId($_POST['user_id']);

	$user_object->setUserLoginStatus('Logout');

	if($user_object->update_user_login_data())
	{
		unset($_SESSION['user_data']);

		session_destroy();

		echo json_encode(['status'=>1]);
	}
}


?>
```

**_src/Chat.php_**

```
<?php

//Chat.php

namespace MyApp;
use Ratchet\MessageComponentInterface;
use Ratchet\ConnectionInterface;
require dirname(__DIR__) . "/database/ChatUser.php";
require dirname(__DIR__) . "/database/ChatRooms.php";

class Chat implements MessageComponentInterface {
    protected $clients;

    public function __construct() {
        $this->clients = new \SplObjectStorage;
        echo 'Server Started';
    }

    public function onOpen(ConnectionInterface $conn) {
        // Store the new connection to send messages to later
        echo 'Server Started';
        $this->clients->attach($conn);

        echo "New connection! ({$conn->resourceId})\n";
    }

    public function onMessage(ConnectionInterface $from, $msg) {
        $numRecv = count($this->clients) - 1;
        echo sprintf('Connection %d sending message "%s" to %d other connection%s' . "\n"
            , $from->resourceId, $msg, $numRecv, $numRecv == 1 ? '' : 's');

        $data = json_decode($msg, true);

        $chat_object = new \ChatRooms;

        $chat_object->setUserId($data['userId']);

        $chat_object->setMessage($data['msg']);

        $chat_object->setCreatedOn(date("Y-m-d h:i:s"));

        $chat_object->save_chat();

        $user_object = new \ChatUser;

        $user_object->setUserId($data['userId']);

        $user_data = $user_object->get_user_data_by_id();

        $user_name = $user_data['user_name'];

        $data['dt'] = date("d-m-Y h:i:s");


        foreach ($this->clients as $client) {
            /*if ($from !== $client) {
                // The sender is not the receiver, send to each client connected
                $client->send($msg);
            }*/

            if($from == $client)
            {
                $data['from'] = 'Me';
            }
            else
            {
                $data['from'] = $user_name;
            }

            $client->send(json_encode($data));
        }
    }

    public function onClose(ConnectionInterface $conn) {
        // The connection is closed, remove it, as we can no longer send it messages
        $this->clients->detach($conn);

        echo "Connection {$conn->resourceId} has disconnected\n";
    }

    public function onError(ConnectionInterface $conn, \Exception $e) {
        echo "An error has occurred: {$e->getMessage()}\n";

        $conn->close();
    }
}

?>
```

**_bin/server.php_**

```
<?php

//server.php

use Ratchet\Server\IoServer;
use Ratchet\Http\HttpServer;
use Ratchet\WebSocket\WsServer;
use MyApp\Chat;

    require dirname(__DIR__) . '/vendor/autoload.php';

    $server = IoServer::factory(
        new HttpServer(
            new WsServer(
                new Chat()
            )
        ),
        8080
    );

    $server->run();


?>
```

**_composer.json_**

```
{
	"autoload": {
        "psr-4": {
            "MyApp\\": "src"
        }
    },
    
    "require": {
        "phpmailer/phpmailer": "^6.2",
        "cboden/ratchet": "v0.4"
    }
}
```
