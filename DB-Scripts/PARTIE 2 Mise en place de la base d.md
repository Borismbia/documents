**PARTIE 2: Mise en place de la base de données**



1. **Script de création de la base des données**



-- Création de la base de données

CREATE DATABASE DATACorp\_DB ;





**2. Script de création des tables**



-- Table Agence

CREATE TABLE Agence (

&nbsp;   id\_agence INT AUTO\_INCREMENT PRIMARY KEY,

&nbsp;   nom VARCHAR(100) NOT NULL,

&nbsp;   ville VARCHAR(100) NOT NULL;





-- Table Client

CREATE TABLE Client (

    id\_client INT AUTO\_INCREMENT PRIMARY KEY,

    id\_agence INT NOT NULL,

    noms VARCHAR(100) NOT NULL,

    prenoms VARCHAR(100) NOT NULL,

&nbsp;   sex ENUM('MALE', 'FEMALE') NOT NULL,

&nbsp;   age INT NOT NULL,

    email VARCHAR(255) UNIQUE NOT NULL,

    telephone VARCHAR(20),

    localisation TEXT,

    CONSTRAINT FK\_Client\_Agence FOREIGN KEY (id\_agence) REFERENCES Agence(id\_agence) ON DELETE RESTRICT,

    INDEX idx\_agence (id\_agence),

    INDEX idx\_nom\_prenom (nom, prenom)

);





-- Table Contrat

CREATE TABLE Contrat (

&nbsp;   id\_contrat INT AUTO\_INCREMENT PRIMARY KEY,

&nbsp;   id\_client INT NOT NULL,

&nbsp;   date\_signature DATE NOT NULL,

&nbsp;   montant\_global DECIMAL(15,2) CHECK (montant\_global >= 0),

&nbsp;   date\_fin\_contrat DATE NOT NULL,

&nbsp;   statut ENUM('ACTIF', 'RESILIE', 'SUSPENDU') DEFAULT 'ACTIF',

&nbsp;   CONSTRAINT FK\_Contrat\_Client FOREIGN KEY (id\_client) REFERENCES Client(id\_client) ON DELETE CASCADE,

&nbsp;   INDEX idx\_client (id\_client),

&nbsp;   INDEX idx\_date\_signature (date\_signature)

) ;



-- Table Transaction

CREATE TABLE Transaction (

&nbsp;   id\_trans INT AUTO\_INCREMENT PRIMARY KEY,

&nbsp;   id\_contrat INT NOT NULL,

&nbsp;   date\_transaction DATETIME DEFAULT CURRENT\_TIMESTAMP,

&nbsp;   montant DECIMAL(12,2) NOT NULL CHECK (montant > 0),

&nbsp;   type\_trans ENUM('DEBIT', 'CREDIT', 'FRAIS') NOT NULL,

&nbsp;   description VARCHAR(255),

&nbsp;   CONSTRAINT FK\_Trans\_Contrat FOREIGN KEY (id\_contrat) REFERENCES Contrat(id\_contrat) ON DELETE CASCADE,

&nbsp;   INDEX idx\_contrat (id\_contrat),

&nbsp;   INDEX idx\_date\_trans (date\_trans)

);





**Creation des Utilisateurs et rôles éventuelles**



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





