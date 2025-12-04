**1. Prérequis** 

**Choix du Système de Gestion de Base de Données (SGBD)**

Le SGBD retenu est MySQL (version 8.0 ou supérieure) 



Héberge l'instance MySQL et la base de données centrale DATACorp\_Central.



**Configuration recommandée** : 4 cœurs CPU, 16 Go RAM, stockage SSD avec minimum 500 Go (selon volumétrie prévisionnelle).



Système d'exploitation : Linux (Ubuntu Server LTS ou RHEL).





**2. Procédures d'installation**



**a. Script de création de la base des données**



-- Création de la base de données

CREATE DATABASE DATACorp\_DB ;





**b. Script de création des tables**



-- Table Agence

CREATE TABLE Agence (

    id\_agence INT AUTO\_INCREMENT PRIMARY KEY,

    nom VARCHAR(100) NOT NULL,

    ville VARCHAR(100) NOT NULL;





-- Table Client

CREATE TABLE Client (

    id\_client INT AUTO\_INCREMENT PRIMARY KEY,

    id\_agence INT NOT NULL,

    noms VARCHAR(100) NOT NULL,

    prenoms VARCHAR(100) NOT NULL,

    sex ENUM('MALE', 'FEMALE') NOT NULL,

    age INT NOT NULL,

    email VARCHAR(255) UNIQUE NOT NULL,

    telephone VARCHAR(20),

    localisation TEXT,

    CONSTRAINT FK\_Client\_Agence FOREIGN KEY (id\_agence) REFERENCES Agence(id\_agence) ON DELETE RESTRICT,

    INDEX idx\_agence (id\_agence),

    INDEX idx\_nom\_prenom (nom, prenom)

);





-- Table Contrat

CREATE TABLE Contrat (

    id\_contrat INT AUTO\_INCREMENT PRIMARY KEY,

    id\_client INT NOT NULL,

    date\_signature DATE NOT NULL,

    montant\_global DECIMAL(15,2) CHECK (montant\_global >= 0),

    date\_fin\_contrat DATE NOT NULL,

    statut ENUM('ACTIF', 'RESILIE', 'SUSPENDU') DEFAULT 'ACTIF',

    CONSTRAINT FK\_Contrat\_Client FOREIGN KEY (id\_client) REFERENCES Client(id\_client) ON DELETE CASCADE,

    INDEX idx\_client (id\_client),

    INDEX idx\_date\_signature (date\_signature)

) ;



-- Table Transaction

CREATE TABLE Transaction (

    id\_trans INT AUTO\_INCREMENT PRIMARY KEY,

    id\_contrat INT NOT NULL,

    date\_transaction DATETIME DEFAULT CURRENT\_TIMESTAMP,

    montant DECIMAL(12,2) NOT NULL CHECK (montant > 0),

    type\_trans ENUM('DEBIT', 'CREDIT', 'FRAIS') NOT NULL,

    description VARCHAR(255),

    CONSTRAINT FK\_Trans\_Contrat FOREIGN KEY (id\_contrat) REFERENCES Contrat(id\_contrat) ON DELETE CASCADE,

    INDEX idx\_contrat (id\_contrat),

    INDEX idx\_date\_trans (date\_trans)

);





**c. Creation des Utilisateurs et rôles éventuelles**



-- Création des rôles

CREATE ROLE 'role\_dg', 'role\_agence', 'role\_audit';



-- Rôle Direction Générale (accès complet en lecture/écriture sur tout le schéma)

GRANT SELECT, INSERT, UPDATE, DELETE ON DATACorp\_Central.\* TO 'role\_dg';

GRANT CREATE TEMPORARY TABLES, EXECUTE ON DATACorp\_Central.\* TO 'role\_dg';



-- Rôle Agence (accès restreint aux données de sa propre agence)

-- Utilisation de Vues ou de Procédures Stockées avec sécurité au niveau ligne (non détaillé ici).

-- Exemple simplifié : accès en lecture/écriture sur les tables mais restriction par application.

GRANT SELECT, INSERT, UPDATE ON DATACorp\_Central.Client TO 'role\_agence';

GRANT SELECT, INSERT, UPDATE ON DATACorp\_Central.Contrat TO 'role\_agence';

GRANT SELECT, INSERT ON DATACorp\_Central.Transaction TO 'role\_agence';

-- Les DELETE sont interdits pour les agences.



-- Rôle Audit (accès en lecture seule sur toutes les tables + table d'audit)

GRANT SELECT ON DATACorp\_Central.\* TO 'role\_audit';

GRANT SELECT, INSERT ON DATACorp\_Central.Audit\_Log TO 'role\_audit';



-- Création des utilisateurs et attribution des rôles

CREATE USER 'user\_dg'@'%' IDENTIFIED BY 'MotDePasseComplexeDG2026!';

CREATE USER 'user\_agence1'@'10.0.1.%' IDENTIFIED BY 'MotDePasseAg1\_2026!';

CREATE USER 'user\_agence2'@'10.0.2.%' IDENTIFIED BY 'MotDePasseAg2\_2026!';

CREATE USER 'user\_agence3'@'10.0.3.%' IDENTIFIED BY 'MotDePasseAg3\_2026!';

CREATE USER 'user\_audit'@'192.168.100.%' IDENTIFIED BY 'MotDePasseAuditSecurise!';



GRANT 'role\_dg' TO 'user\_dg'@'%';

GRANT 'role\_agence' TO 'user\_agence1'@'10.0.1.%';

GRANT 'role\_agence' TO 'user\_agence2'@'10.0.2.%';

GRANT 'role\_agence' TO 'user\_agence3'@'10.0.3.%';

GRANT 'role\_audit' TO 'user\_audit'@'192.168.100.%';





**3. Comment lancer les procédures stockées de test**



Connectez-vous à votre base de données et ouvrez une nouvelle fenêtre de requête;



Utilisez l'instruction EXEC (SQL Server) ou CALL (MySQL) pour appeler la procédure, en spécifiant son nom et ses paramètres. Exemple pour MySQL : CALL NomDeLaProcedure('valeur1', 123);



Exécutez la requête pour lancer le test. 





**4. Problèmes connus/limites**



Les principaux problèmes et limites de la gestion de bases de données avec SQL incluent la rigidité de son schéma, des problèmes d'évolutivité liés à son architecture, des limitations dans la gestion des données non structurées, et des difficultés de performance sur des ensembles de données.







