# -todo-cli-python
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_DESC_LEN 100

/* ---------- Node structure ---------- */
typedef struct Task {
    int id;
    char description[MAX_DESC_LEN];
    struct Task *next;
} Task;

/* ---------- Globals ---------- */
Task *head = NULL;      /* head of the linked list */
int nextId = 1;         /* auto-incrementing unique ID generator */

/* ---------- Function Prototypes ---------- */
void showMenu(void);
void insertTask(void);
void deleteTask(void);
void updateTask(void);
void displayTasks(void);
Task* findTask(int id);
void clearInputBuffer(void);
void freeAllTasks(void);

int main(void) {
    int choice;

    printf("=====================================\n");
    printf("   TO-DO LIST MANAGEMENT SYSTEM\n");
    printf("=====================================\n");

    do {
        showMenu();
        printf("Enter your choice: ");

        /* Validate menu input */
        if (scanf("%d", &choice) != 1) {
            printf("\nInvalid input! Please enter a number (1-5).\n\n");
            clearInputBuffer();
            continue;
        }
        clearInputBuffer();

        switch (choice) {
            case 1: insertTask();   break;
            case 2: deleteTask();   break;
            case 3: updateTask();   break;
            case 4: displayTasks(); break;
            case 5:
                printf("\nExiting To-Do List Manager. Goodbye!\n");
                break;
            default:
                printf("\nInvalid choice! Please select an option between 1 and 5.\n");
        }
        printf("\n");

    } while (choice != 5);

    freeAllTasks();
    return 0;
}

/* ---------- Display Menu ---------- */
void showMenu(void) {
    printf("-------------------------------------\n");
    printf("1. Insert Task\n");
    printf("2. Delete Task\n");
    printf("3. Update Task\n");
    printf("4. Display Tasks\n");
    printf("5. Exit\n");
    printf("-------------------------------------\n");
}

/* ---------- Clear leftover input (e.g., newline) ---------- */
void clearInputBuffer(void) {
    int c;
    while ((c = getchar()) != '\n' && c != EOF);
}

/* ---------- Insert a new task ---------- */
void insertTask(void) {
    Task *newTask = (Task *) malloc(sizeof(Task));
    if (newTask == NULL) {
        printf("Error: Memory allocation failed.\n");
        return;
    }

    printf("Enter task description: ");
    if (fgets(newTask->description, MAX_DESC_LEN, stdin) == NULL) {
        printf("Error reading task description.\n");
        free(newTask);
        return;
    }
    /* Remove trailing newline captured by fgets */
    newTask->description[strcspn(newTask->description, "\n")] = '\0';

    /* Reject empty descriptions */
    if (strlen(newTask->description) == 0) {
        printf("Error: Task description cannot be empty. Task not added.\n");
        free(newTask);
        return;
    }

    newTask->id = nextId++;
    newTask->next = NULL;

    /* Append at the end of the list to preserve insertion order */
    if (head == NULL) {
        head = newTask;
    } else {
        Task *temp = head;
        while (temp->next != NULL) {
            temp = temp->next;
        }
        temp->next = newTask;
    }

    printf("Success: Task added with ID %d.\n", newTask->id);
}

/* ---------- Find a task by ID (returns NULL if not found) ---------- */
Task* findTask(int id) {
    Task *temp = head;
    while (temp != NULL) {
        if (temp->id == id) {
            return temp;
        }
        temp = temp->next;
    }
    return NULL;
}

/* ---------- Delete a task by ID ---------- */
void deleteTask(void) {
    if (head == NULL) {
        printf("Task list is empty. Nothing to delete.\n");
        return;
    }

    int id;
    printf("Enter Task ID to delete: ");
    if (scanf("%d", &id) != 1) {
        printf("Error: Invalid Task ID input.\n");
        clearInputBuffer();
        return;
    }
    clearInputBuffer();

    if (id <= 0) {
        printf("Error: Task ID must be a positive number.\n");
        return;
    }

    Task *curr = head;
    Task *prev = NULL;

    while (curr != NULL && curr->id != id) {
        prev = curr;
        curr = curr->next;
    }

    if (curr == NULL) {
        printf("Error: Task with ID %d does not exist.\n", id);
        return;
    }

    if (prev == NULL) {
        head = curr->next;   /* deleting the head node */
    } else {
        prev->next = curr->next;
    }

    printf("Success: Task ID %d (\"%s\") deleted.\n", curr->id, curr->description);
    free(curr);
}

/* ---------- Update an existing task ---------- */
void updateTask(void) {
    if (head == NULL) {
        printf("Task list is empty. Nothing to update.\n");
        return;
    }

    int id;
    printf("Enter Task ID to update: ");
    if (scanf("%d", &id) != 1) {
        printf("Error: Invalid Task ID input.\n");
        clearInputBuffer();
        return;
    }
    clearInputBuffer();

    Task *taskToUpdate = findTask(id);
    if (taskToUpdate == NULL) {
        printf("Error: Task with ID %d does not exist.\n", id);
        return;
    }

    char newDesc[MAX_DESC_LEN];
    printf("Enter new description for Task ID %d (current: \"%s\"): ",
           taskToUpdate->id, taskToUpdate->description);

    if (fgets(newDesc, MAX_DESC_LEN, stdin) == NULL) {
        printf("Error reading new description.\n");
        return;
    }
    newDesc[strcspn(newDesc, "\n")] = '\0';

    if (strlen(newDesc) == 0) {
        printf("Error: New description cannot be empty. Task not updated.\n");
        return;
    }

    strncpy(taskToUpdate->description, newDesc, MAX_DESC_LEN - 1);
    taskToUpdate->description[MAX_DESC_LEN - 1] = '\0';

    printf("Success: Task ID %d updated.\n", taskToUpdate->id);
}

/* ---------- Display all tasks ---------- */
void displayTasks(void) {
    if (head == NULL) {
        printf("Task list is empty. No tasks to display.\n");
        return;
    }

    printf("\n%-10s %-s\n", "Task ID", "Description");
    printf("-------------------------------------\n");

    Task *temp = head;
    while (temp != NULL) {
        printf("%-10d %-s\n", temp->id, temp->description);
        temp = temp->next;
    }
}

/* ---------- Free all remaining nodes before program exit ---------- */
void freeAllTasks(void) {
    Task *temp;
    while (head != NULL) {
        temp = head;
        head = head->next;
        free(temp);
    }
}
