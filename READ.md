#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <sys/wait.h>
#include <sys/types.h>

#define WELCOME_MSG "Bienvenue dans le Shell ENSEA.\nPour quitter, tapez 'exit'.\n"
#define PROMPT "enseash%"
#define EXIT "Bye bye..."

int main() {
    int status;
    int exitstatus;
    int exitsignal;

    /* Buffer for storing the user's command */
    char command[100];

    /* Print the welcome message */
    write(1, WELCOME_MSG, strlen(WELCOME_MSG));

    /* Loop indefinitely to accept commands */
    while (1) {
        /* Print the shell prompt */
        write(1, PROMPT, strlen(PROMPT));

        /* Read user input */
        int bytes_read = read(0, command, 100);
        command[bytes_read - 1] = '\0';  // Remove the newline character

        /* Check if the command is "exit" */
        if (strcmp(command, "exit") == 0) {
            /* Exit the loop and the program */
            write(1, EXIT, strlen(EXIT));
            exit(0);
        } else {
            /* Fork to create a child process */
            pid_t pid = fork();

            /* Child process */
            if (pid == 0) {
                execlp(command, command, (char *)NULL);

                // The following code will be executed only if execlp fails
                perror("Error");  // Print an error message
                exit(1);          // Exit with an error code
            } else {
                /* Parent process */
                /* Wait for the child process to finish */
                wait(&status);

                if (WIFEXITED(status)) {
                    exitstatus = WEXITSTATUS(status);
                    write(1, " [exit:", 7);
                    char status_str[4];
                    snprintf(status_str, sizeof(status_str), "%d", exitstatus);
                    write(1, status_str, strlen(status_str));
                    write(1, "] % ", 4);
                } else if (WIFSIGNALED(status)) {
                    exitsignal = WTERMSIG(status);
                    write(1, " [sign:", 7);
                    char signal_str[4];
                    snprintf(signal_str, sizeof(signal_str), "%d", exitsignal);
                    write(1, signal_str, strlen(signal_str));
                    write(1, "] % ", 4);
                }
            }
        }
    }
    return 0;
}

