#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define N 10
#define FILENAME_CLIENT "client.txt"
#define FILENAME_COMPTE "compte.txt"

typedef struct Date {
    int Jour;
    int Mois;
    int Annee;
} Date;

typedef struct Clientbank {
    int Code_cli;
    char Nom[N];
    char Prenom[N];
} Clientbank;

typedef struct Comptebank {
    int code_cpt;
    int Code_cli;
    int somme;
    Date d_compte;
} Comptebank;
void sousMenuClients(FILE *clientFile);
void sousMenuComptes(FILE *compteFile);
void sousMenuOperations(FILE *compteFile);
void displayClient(const Clientbank *client) {
    printf("\nCode client : %d", client->Code_cli);
    printf("\nNom : %s | Prenom : %s", client->Nom, client->Prenom);
    printf("\n");
}

void addClient(FILE *file) {
    Clientbank client;
    printf("Code Client : ");
    scanf("%d", &client.Code_cli);

    printf("Nom : ");
    scanf("%s", client.Nom);

    printf("Prenom : ");
    scanf("%s", client.Prenom);

    fwrite(&client, sizeof(client), 1, file);
    printf("Client ajoute avec succes.\n");
}

void displayAllClients(FILE *file) {
    Clientbank client;
    rewind(file);

    while (fread(&client, sizeof(Clientbank), 1, file)) {
        displayClient(&client);
    }
}

void modifyClient(FILE *file) {
    int code;
    printf("Entrez le code du client a modifier : ");
    scanf("%d", &code);

    Clientbank client;
    FILE *tempFile = fopen("temp.txt", "w");

    while (fread(&client, sizeof(Clientbank), 1, file)) {
        if (client.Code_cli == code) {
            printf("Nouveau nom : ");
            scanf("%s", client.Nom);
            printf("Nouveau prenom : ");
            scanf("%s", client.Prenom);
        }
        fwrite(&client, sizeof(Clientbank), 1, tempFile);
    }

    fclose(file);
    fclose(tempFile);

    remove(FILENAME_CLIENT);
    rename("temp.txt", FILENAME_CLIENT);
    file = fopen(FILENAME_CLIENT, "a+");
    printf("Client modifie avec succes.\n");
}

void deleteClient(FILE *file) {
    int code;
    printf("Entrez le code du client a supprimer : ");
    scanf("%d", &code);

    Clientbank client;
    FILE *tempFile = fopen("temp.txt", "w");

    int found = 0;

    while (fread(&client, sizeof(Clientbank), 1, file)) {
        if (client.Code_cli == code) {
            found = 1;
            printf("Client supprime avec succes.\n");
        } else {
            fwrite(&client, sizeof(Clientbank), 1, tempFile);
        }
    }

    fclose(file);
    fclose(tempFile);

    remove(FILENAME_CLIENT);
    rename("temp.txt", FILENAME_CLIENT);
    file = fopen(FILENAME_CLIENT, "a+");

    if (!found) {
        printf("Client non trouve.\n");
    }
}

void addAccount(FILE *file) {
    Comptebank compte;
    printf("Code Compte : ");
    scanf("%d", &compte.code_cpt);

    printf("Code Client : ");
    scanf("%d", &compte.Code_cli);

    printf("Date de creation du compte | ");
    printf("Jour : ");
    scanf("%d", &compte.d_compte.Jour);

    printf("Mois : ");
    scanf("%d", &compte.d_compte.Mois);

    printf("Annee : ");
    scanf("%d", &compte.d_compte.Annee);

    printf("Somme : ");
    scanf("%d", &compte.somme);

    fwrite(&compte, sizeof(compte), 1, file);
    printf("Compte ajoute avec succes.\n");
}

void displayAllAccounts(FILE *file) {
    Comptebank compte;
    rewind(file);

    while (fread(&compte, sizeof(Comptebank), 1, file)) {
        printf("\nCode compte : %d", compte.code_cpt);
        printf("\nCode client : %d", compte.Code_cli);
        printf("\nDate creation %d / %d / %d", compte.d_compte.Jour, compte.d_compte.Mois, compte.d_compte.Annee);
        printf("\nSomme %d", compte.somme);
        printf("\n-----------------------------------------");
    }
}

void deleteAccount(FILE *file) {
    int code;
    printf("Entrez le code du compte a supprimer : ");
    scanf("%d", &code);

    Comptebank compte;
    FILE *tempFile = fopen("temp_compte.txt", "w");

    int found = 0;

    while (fread(&compte, sizeof(Comptebank), 1, file)) {
        if (compte.code_cpt == code) {
            found = 1;
            printf("Compte supprime avec succes.\n");
        } else {
            fwrite(&compte, sizeof(Comptebank), 1, tempFile);
        }
    }

    fclose(file);
    fclose(tempFile);

    remove(FILENAME_COMPTE);
    rename("temp_compte.txt", FILENAME_COMPTE);
    file = fopen(FILENAME_COMPTE, "a+");

    if (!found) {
        printf("Compte non trouve.\n");
    }
}

void withdrawal(FILE *file) {
    int code;
    printf("Entrez le code du compte pour le retrait : ");
    scanf("%d", &code);

    Comptebank compte;
    FILE *tempFile = fopen("temp_compte.txt", "w");

    int found = 0;

    while (fread(&compte, sizeof(Comptebank), 1, file)) {
        if (compte.code_cpt == code) {
            found = 1;
            int withdrawalAmount;
            printf("Montant à retirer : ");
            scanf("%d", &withdrawalAmount);

            if (withdrawalAmount > 0 && withdrawalAmount <= compte.somme) {
                compte.somme -= withdrawalAmount;
                printf("Retrait de %d DT effectué avec succes.\n", withdrawalAmount);
            } else {
                printf("Montant invalide ou solde insuffisant.\n");
            }
        }
        fwrite(&compte, sizeof(Comptebank), 1, tempFile);
    }

    fclose(file);
    fclose(tempFile);

    remove(FILENAME_COMPTE);
    rename("temp_compte.txt", FILENAME_COMPTE);
    file = fopen(FILENAME_COMPTE, "a+");

    if (!found) {
        printf("Compte non trouvé.\n");
    }
}


void deposit(FILE *file) {
    int code;
    printf("Entrez le code du compte pour le versement : ");
    scanf("%d", &code);

    Comptebank compte;
    FILE *tempFile = fopen("temp_compte.txt", "w");

    int found = 0;

    while (fread(&compte, sizeof(Comptebank), 1, file)) {
        if (compte.code_cpt == code) {
            found = 1;
            int depositAmount;
            printf("Montant a verser : ");
            scanf("%d", &depositAmount);

            if (depositAmount > 0) {
                compte.somme += depositAmount;
                printf("Versement de %d DT effectué avec succes.\n", depositAmount);
            } else {
                printf("Montant invalide.\n");
            }
        }
        fwrite(&compte, sizeof(Comptebank), 1, tempFile);
    }

    fclose(file);
    fclose(tempFile);

    remove(FILENAME_COMPTE);
    rename("temp_compte.txt", FILENAME_COMPTE);
    file = fopen(FILENAME_COMPTE, "a+");

    if (!found) {
        printf("Compte non trouve.\n");
    }
}


void menu(FILE *clientFile, FILE *compteFile) {
    int choix;
    printf("\n----Bienvenue----\n");
    do {
        printf("1- Gestion des clients\n");
        printf("2- Gestion des comptes\n");
        printf("3- Gestion des operations\n");
        printf("4- Quitter le programme\n");
        printf("Choisir le menu : ");
        scanf("%d", &choix);

        switch (choix) {
            case 1:
                sousMenuClients(clientFile);
                break;
            case 2:
                sousMenuComptes(compteFile);
                break;
            case 3:
                sousMenuOperations(compteFile);
                break;
            case 4:
                printf("Au revoir !\n");
                break;
            default:
                printf("Choix invalide. Veuillez reessayer.\n");
        }

    } while (choix != 4);
}

void sousMenuClients(FILE *clientFile) {
    int choix;
    do {
        printf("1- Ajouter\n");
        printf("2- Modifier\n");
        printf("3- Supprimer\n");
        printf("4- Afficher\n");
        printf("5- Retour\n");
        printf("Choisir un sous-menu : ");
        scanf("%d", &choix);

        switch (choix) {
            case 1:
                addClient(clientFile);
                break;
            case 2:
                modifyClient(clientFile);
                break;
            case 3:
                deleteClient(clientFile);
                break;
            case 4:
                displayAllClients(clientFile);
                break;
            case 5:
                printf("Retour au menu principal.\n");
                break;
            default:
                printf("Choix invalide. Veuillez reessayer.\n");
        }

    } while (choix != 5);
}

void sousMenuComptes(FILE *compteFile) {
    int choix;
    do {
        printf("1- Ajouter\n");
        printf("2- Rechercher\n");
        printf("3- Afficher la liste\n");
        printf("4- Supprimer\n");
        printf("5- Retour\n");
        printf("Choisir un sous-menu : ");
        scanf("%d", &choix);

        switch (choix) {
            case 1:
                addAccount(compteFile);
                break;
            case 2:
                // Ajouter le code pour rechercher un compte
                break;
            case 3:
                displayAllAccounts(compteFile);
                break;
            case 4:
                deleteAccount(compteFile);
                break;
            case 5:
                printf("Retour au menu principal.\n");
                break;
            default:
                printf("Choix invalide. Veuillez reessayer.\n");
        }

    } while (choix != 5);
}

void sousMenuOperations(FILE *compteFile) {
    int choix;
    do {
        printf("1- Retrait\n");
        printf("2- Versement d'argent\n");
        printf("3- Retour\n");
        printf("Choisir un sous-menu : ");
        scanf("%d", &choix);

        switch (choix) {
            case 1:
                withdrawal(compteFile);
                break;
            case 2:
                deposit(compteFile);
                break;
            case 3:
                printf("Retour au menu principal.\n");
                break;
            default:
                printf("Choix invalide. Veuillez reessayer.\n");
        }

    } while (choix != 3);
}

int main() {
    FILE *clientFile = fopen(FILENAME_CLIENT, "a+");
    FILE *compteFile = fopen(FILENAME_COMPTE, "a+");

    if (clientFile == NULL || compteFile == NULL) {
        printf("Erreur lors de l'ouverture des fichiers.\n");
    }

    menu(clientFile, compteFile);

    fclose(clientFile);
    fclose(compteFile);

    return 0;
}
