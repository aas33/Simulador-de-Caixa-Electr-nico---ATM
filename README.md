# Simulador-de-Caixa-Electrónico -- ATM


#include <time.h>
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <string.h>
#include <conio.h> // Esta biblioteca é específica para Windows

// Definindo a estrutura de uma conta bancária
struct Conta {
    char username[20];
    char pin[5]; // A senha terá apenas 4 dígitos
    float saldo;
};

// Estrutura para representar uma transação
struct Transacao {
    char remetente[20];
    char destinatario[20];
    float valor;
};

// Função para realizar login
bool login(struct Conta contas[], int numContas, char username[], char pin[]) {
    for (int i = 0; i < numContas; i++) {
        if (strcmp(contas[i].username, username) == 0 && strcmp(contas[i].pin, pin) == 0) {
            return true;
        }
    }
    return false;
}

// Função para consultar saldo
void consultarSaldo(struct Conta contas[], int numContas, char username[]) {
    for (int i = 0; i < numContas; i++) {
        if (strcmp(contas[i].username, username) == 0) {
            printf("\033[1;34mSaldo atual: %.2f\n\033[0m", contas[i].saldo);
            return;
        }
    }
    printf("Conta não encontrada.\n");
}

// Função para realizar saque
void sacar(struct Conta contas[], int numContas, char username[], float valor) {
    for (int i = 0; i < numContas; i++) {
        if (strcmp(contas[i].username, username) == 0) {
            if (contas[i].saldo >= valor) {
                contas[i].saldo -= valor;
                printf("\033[1;31mSaque de %.2f realizado com sucesso.\n\033[0m", valor);
                printf("\033[1;34mSaldo atual: %.2f\n\033[0m", contas[i].saldo);
            } else {
                printf("\033[1;31mSaldo insuficiente.\n\033[0m");
            }
            return;
        }
    }
    printf("Conta não encontrada.\n");
}

// Função para realizar depósito
void depositar(struct Conta contas[], int numContas, char username[], float valor) {
    for (int i = 0; i < numContas; i++) {
        if (strcmp(contas[i].username, username) == 0) {
            contas[i].saldo += valor;
            printf("\033[1;32mDepósito de %.2f realizado com sucesso.\n\033[0m", valor);
            printf("\033[1;34mSaldo atual: %.2f\n\033[0m", contas[i].saldo);
            return;
        }
    }
    printf("Conta não encontrada.\n");
}

// Função para realizar transferência entre contas
void transferir(struct Conta contas[], int numContas, struct Transacao transacao) {
    bool remetenteEncontrado = false;
    bool destinatarioEncontrado = false;

    for (int i = 0; i < numContas; i++) {
        if (strcmp(contas[i].username, transacao.remetente) == 0) {
            remetenteEncontrado = true;
            if (contas[i].saldo >= transacao.valor) {
                contas[i].saldo -= transacao.valor;
            } else {
                printf("\033[1;31mSaldo insuficiente para transferência.\n\033[0m");
                return;
            }
        }
        if (strcmp(contas[i].username, transacao.destinatario) == 0) {
            destinatarioEncontrado = true;
            contas[i].saldo += transacao.valor;
        }
    }

    if (!remetenteEncontrado || !destinatarioEncontrado) {
        printf("\033[1;31mConta de remetente ou destinatário não encontrada.\n\033[0m");
    } else {
        printf("\033[1;32mTransferência de %.2f realizada com sucesso de %s para %s.\n\033[0m", transacao.valor, transacao.remetente, transacao.destinatario);
    }
}

// Função para limpar a tela do console
void limparTela() {
    system("cls || clear");
}

int main() {
    // Criando algumas contas de exemplo
    struct Conta contas[3] = {{"user1", "1234", 1000.0},
                               {"user2", "5678", 500.0},
                               {"user3", "9012", 200.0}};
    int numContas = 3;

    while (true) {
        // Simulação de login
        char username[20];
        char pin[5]; // A senha terá apenas 4 dígitos
        printf("Digite o nome de usuário: ");
        scanf("%s", username);
        printf("Digite o PIN (4 dígitos): ");
        int i = 0;
        char ch;
        while ((ch = getch()) != '\r' && i < 4) {
            if (ch == '\b' && i > 0) {
                printf("\b \b"); // Apaga o caractere digitado
                i--;
                continue;
            }
            if (ch == '\b' && i == 0) {
                continue; // Ignora backspace se o PIN estiver vazio
            }
            pin[i++] = ch;
            printf("*"); // Exibe asterisco em vez do caractere digitado
        }
        if (i < 4) {
            printf("PIN deve ter exatamente 4 dígitos.\n");
            limparTela(); // Limpa a tela antes de voltar para o login
            continue; // Solicita novamente o PIN
        }
        pin[i] = '\0'; // Adiciona o caractere nulo ao final do PIN
        printf("\n");

        if (login(contas, numContas, username, pin)) {
            printf("Login bem-sucedido!\n");
            limparTela();

            // Menu principal após o login
            int opcao;
            float valor;
            struct Transacao transacao;
            do {
                printf("\nMenu:\n");
                printf("1. Consultar saldo\n");
                printf("2. Sacar\n");
                printf("3. Depositar\n");
                printf("4. Transferir\n");
                printf("5. Sair\n");
                printf("Escolha uma opção: ");
                scanf("%d", &opcao);

                switch (opcao) {
                    case 1:
                        consultarSaldo(contas, numContas, username);
                        break;
                    case 2:
                        printf("Digite o valor a ser sacado: ");
                        scanf("%f", &valor);
                        sacar(contas, numContas, username, valor);
                        break;
                    case 3:
                        printf("Digite o valor a ser depositado: ");
                        scanf("%f", &valor);
                        depositar(contas, numContas, username, valor);
                        break;
                    case 4:
                        printf("Digite o nome de usuário do destinatário: ");
                        scanf("%s", transacao.destinatario);
                        printf("Digite o valor a ser transferido: ");
                        scanf("%f", &transacao.valor);
                        strcpy(transacao.remetente, username);
                        transferir(contas, numContas, transacao);
                        break;
                    case 5:
                        printf("Saindo...\n");
                        break;
                    default:
                        printf("Opção inválida. Tente novamente.\n");
                }
            } while (opcao != 5);

            limparTela(); // Limpa a tela antes de voltar para o login
        } else {
            printf("Nome de usuário ou PIN incorretos.\n");
            limparTela(); // Limpa a tela antes de voltar para o login
        }
    }

    return 0;
}
