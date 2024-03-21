# Découverte des Sockets avec Java

La notion de sockets a été introduite dans les distributions de Berkeley (un fameux système de type UNIX, dont beaucoup de distributions actuelles utilisent des morceaux de code), c'est la raison pour laquelle on parle parfois de sockets BSD (Berkeley Software Distribution).

Il s'agit d'un modèle permettant la communication inter processus (IPC - Inter Process Communication) afin de permettre à divers processus de communiquer aussi bien sur une même machine qu'à travers un réseau TCP/IP.

La communication par socket est souvent comparée aux communications humaines. On distingue ainsi deux modes de communication :

+ Le mode connecté (comparable à une communication téléphonique), utilisant le protocole TCP. Dans ce mode de communication, une connexion durable est établie entre les deux processus, de telle façon que l'adresse de destination n'est pas nécessaire à chaque envoi de données.
+ Le mode non connecté (analogue à une communication par courrier), utilisant le protocole UDP. Ce mode nécessite l'adresse de destination à chaque envoi, et aucun accusé de réception n'est donné.

Depuis son origine, Java fournit plusieurs classes et interfaces destinées à faciliter l'utilisation du réseau par programmation en reposant sur les sockets. Celles-ci peuvent être mises en oeuvre pour réaliser des échanges utilisant le protocole réseau IP avec les protocoles de transport TCP ou UDP. Les échanges se font entre deux parties : un client et un serveur.

Nous allons étudier ici les sockets TCP.

## Manipuler une adresse IP

Le package java.net de la plate-forme Java fournit une classe InetAddress qui nous permet de récupérer et manipuler son adresse ip. Cette classe n'a pas de constructeur, pour pouvoir avoir une instance de cette classe on a besoin des méthodes de classe. Voici les méthodes :

+ getLocalHost() : elle retourne un objet qui contient l'adresse IP de la machine locale.

+ getByName(String nom_de_l_machine) : elle retourne un objet qui contient l'adresse IP de la machine dont le nom est passé en paramètre.

+ getAllByName(String nom_de_l_machine) : elle retourne un tableau d'objets qui contient l'ensemble d'adresses IP de la machine qui correspond au nom passé en paramètre.

+ A présent, voyons les méthodes applicables à un objet de cette classe :

+ getHostName() : elle retourne le nom de la machine dont l'adresse est stockée dans l'objet.

+ getAddress() : elle retourne l'adresse IP stockée dans l'objet sous forme d'un tableau de 4 octets.

+ toString() : elle retourne un String qui correspond au nom de la machine et son adresse.

### Un exemple de manipulation d'adresse : 

    public static void main(String[] zero) {

        InetAddress LocaleAdresse ;
        InetAddress ServeurAdresse;

        try {

            LocaleAdresse = InetAddress.getLocalHost();
            System.out.println("L'adresse locale est : "+LocaleAdresse );

            ServeurAdresse= InetAddress.getByName("tf1.fr");
            System.out.println("L'adresse du serveur de TF1 est : "+ServeurAdresse);

        } catch (UnknownHostException e) {

            e.printStackTrace();
        }
    }

## Le socket

Le socket peut être vu comme une sorte de connecteur sur lequel on va venir se greffer pour communiquer. Il écoute sur un port spécifique et capture les connexions venant de brancher à lui.

Ici, le code d'un serveur créant un socket qui va se mettre en écoute de clients : 

    public static void main(String[] args) {

        ServerSocket socketserver  ;
        Socket socketduserveur ;

        try {

            socketserver = new ServerSocket(2024);
            socketduserveur = socketserver.accept();
            System.out.println("Un client s'est connecté !");
            socketserver.close();
            socketduserveur.close();

        }catch (IOException e) {
            e.printStackTrace();
        }
    }

Le socket est créé sur un port (2024) et il accepte les connexions avec la méthode accept(). Celle-ci bloque toute l'exécution jusqu'à ce qu'un client tente de se connecter au socket.
Une fois contacté par un client, un message est affiché et le socket refermé.

Du coté d'un client, le code suivant se charge de joindre le serveur : 

    public static void main(String[] args) {

        Socket socket;

        try {

            socket = new Socket(InetAddress.getLocalHost(),2024);
            socket.close();

        }catch (UnknownHostException e) {

            e.printStackTrace();
        }catch (IOException e) {

            e.printStackTrace();
        }
    }

Ici, on crée un socket sur le port dédié de notre serveur et on referme celui-ci. Rien de très intéressant mais cela va vous permettre de voir l'échange entre le client et le serveur. 

Lancez le serveur puis lancez le programme client et observez l'échange se produire.

## S'envoyer des messages

Nous avons pu établir une connexion entre un client et un serveur avec des sockets. Maintenant, nous allons voir comment échanger des messages entre les deux. Modifions le serveur en ajoutant cela : 

    public static void main(String[] args) {

        ServerSocket socketserver  ;
        Socket socketduserveur ;
        BufferedReader in;
        PrintWriter out;

        try {

            socketserver = new ServerSocket(2024);
            System.out.println("Le serveur est à l'écoute du port "+socketserver.getLocalPort());
            socketduserveur = socketserver.accept();
            System.out.println("Un client s'est connecté");
            out = new PrintWriter(socketduserveur.getOutputStream());
            out.println("Vous êtes connecté en tant que client !");
            out.flush();

            socketduserveur.close();
            socketserver.close();

        }catch (IOException e) {

            e.printStackTrace();
        }
    }

Lorsqu'un client se connecte, on va utiliser le socket pour récupérer un objet PrintWriter qui nous permettra d'envoyer des données vers le client. On écrira dans cet objet comme on peut le faire dans la console en appelant sur lui la méthode println.
Enfin, la méthode flush appelée permettra de valider la transmission des données dans le flux.

Coté client à présent, observons le code : 

    public static void main(String[] args) {


        Socket socket;
        BufferedReader in;
        PrintWriter out;

        try {

            socket = new Socket(InetAddress.getLocalHost(),2024);
            System.out.println("Demande de connexion");

            in = new BufferedReader (new InputStreamReader (socket.getInputStream()));
            String message_distant = in.readLine();
            System.out.println(message_distant);

            socket.close();

        }catch (UnknownHostException e) {

            e.printStackTrace();
        }catch (IOException e) {

            e.printStackTrace();
        }
    }

Nous avons besoin ici de lire le flux reçu de notre serveur. Pour cela, on va récupérer un BufferedReader sur le socket dans lequel on va aller lire les données reçues.

## Notion de thread

Les versions actuelles de nos exemples posent un petit problème : une fois qu'un client est connecté, il n'est plus possible d'accepter de nouvelles connexions. Si on conçoit une application de chat en ligne par exemple, ceci aurait pour conséquence de ne pouvoir enegager dans la discussion qu'un seul utilisateur... problématique.
Ce problème peut être facilement résolu à l'aide des threads (processus légers). Ces threads sont des processus qui vont pouvoir être lancés en paralèlle afin de ne pas occuper un seul processus principal et ainsi de pouvoir gérer simultanément plusieurs connexions.

Observez ce nouveau code de serveur :

    public class Serveur {
    
        public static void main(String[] args){
            
            ServerSocket socket;
            try {
            socket = new ServerSocket(2024);
            Thread t = new Thread(new Accepter_clients(socket));
            t.start();
            System.out.println("Mes employeurs sont prêts !");
            
            } catch (IOException e) {
                
                e.printStackTrace();
            }
        }
    }

    class Accepter_clients implements Runnable {

	   private ServerSocket socketserver;
	   private Socket socket;
	   private int nbrclient = 1;
		public Accepter_clients(ServerSocket s){
			socketserver = s;
		}
		
		public void run() {

	        try {
	        	while(true){
			  socket = socketserver.accept(); // Un client se connecte on l'accepte
	                  System.out.println("Le client numéro "+nbrclient+ " est connecté !");
	                  nbrclient++;
	                  socket.close();
	        	}
	        
	        } catch (IOException e) {
				e.printStackTrace();
			}
		}

	}

Nous avons une nouvelle classe qui implémente une interface Runnable qui permet d'accéder aux fonctionnalités des threads. Cette classe est passée en paramètre d'une nouvelle instance de Thread et lorsqu'on appelera sa méthode start(), le code de la méthode run() sera exécuté.
C'est alors ici que la gestion de la connexion du client se produira. Dans une boucle infinie (while (true)), on pourra donc y accepter de multiples connexions de clients.

Coté client, on reste sur du classique comme on a pu le voir avant : 

    public static void main(String[] args){

		Socket socket;
		try {
		socket = new Socket("localhost",2024);
		socket.close();
		} catch (IOException e) {
			
			e.printStackTrace();
		}
	}

## Quelques exercices à présent

### 1. Chat public

Proposez une solution afin de pouvoir chatter entre personnes connectées tous ensemble. Tout le monde verra les messages de tout le monde.

### 2. Chat ciblé

Permettez maintenant d'avoir un menu coté client ou on choisira le nom de la personne avec qui parler (une personne en particulier en saisissant son nom ou all si on veut parler à tous les connectés).

### 3. Diffusion de l'historique des messages

Lorsqu'un utilisateur se connectera en cours de conversation, on lui affichera l'intégralité des discussions publiques depuis le lancement du serveur.

### 4. Création de salons

On pourra ajouter un salon de discussion et y inviter d'autres utilisateurs. Seuls les membres d'un salon pourront en voir les messages.

### 5. Authentification des utilisateurs

Lors de la connexion, le serveur demander un login et un mot de passe à l'utilisateur et vérifiera dans un fichier contenant les logins/mots de passe si son identité est valide.

