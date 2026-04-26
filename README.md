#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// ---------------- STRUCTURE ----------------
typedef struct Node {
    int id;
    char name[50];
    int age;
    char disease[50];
    struct Node *left;
    struct Node *right;
    int height;
} Node;

// ---------------- AVL FUNCTIONS ----------------
int height(Node *n) {
    return (n == NULL) ? 0 : n->height;
}

int max(int a, int b) {
    return (a > b) ? a : b;
}

Node* createNode(int id, char name[], int age, char disease[]) {
    Node* node = (Node*)malloc(sizeof(Node));
    node->id = id;
    strcpy(node->name, name);
    node->age = age;
    strcpy(node->disease, disease);
    node->left = node->right = NULL;
    node->height = 1;
    return node;
}

Node* rightRotate(Node *y) {
    Node *x = y->left;
    Node *T2 = x->right;

    x->right = y;
    y->left = T2;

    y->height = max(height(y->left), height(y->right)) + 1;
    x->height = max(height(x->left), height(x->right)) + 1;

    return x;
}

Node* leftRotate(Node *x) {
    Node *y = x->right;
    Node *T2 = y->left;

    y->left = x;
    x->right = T2;

    x->height = max(height(x->left), height(x->right)) + 1;
    y->height = max(height(y->left), height(y->right)) + 1;

    return y;
}

int getBalance(Node *n) {
    return (n == NULL) ? 0 : height(n->left) - height(n->right);
}

// ---------------- INSERT ----------------
Node* insert(Node* node, int id, char name[], int age, char disease[]) {
    if (node == NULL)
        return createNode(id, name, age, disease);

    if (id < node->id)
        node->left = insert(node->left, id, name, age, disease);
    else if (id > node->id)
        node->right = insert(node->right, id, name, age, disease);
    else {
        printf("❌ Duplicate ID not allowed!\n");
        return node;
    }

    node->height = 1 + max(height(node->left), height(node->right));

    int balance = getBalance(node);

    // LL
    if (balance > 1 && id < node->left->id)
        return rightRotate(node);

    // RR
    if (balance < -1 && id > node->right->id)
        return leftRotate(node);

    // LR
    if (balance > 1 && id > node->left->id) {
        node->left = leftRotate(node->left);
        return rightRotate(node);
    }

    // RL
    if (balance < -1 && id < node->right->id) {
        node->right = rightRotate(node->right);
        return leftRotate(node);
    }

    return node;
}

// ---------------- SEARCH ----------------
Node* search(Node* root, int id) {
    if (root == NULL || root->id == id)
        return root;

    if (id < root->id)
        return search(root->left, id);

    return search(root->right, id);
}

// ---------------- MIN VALUE NODE ----------------
Node* minValueNode(Node* node) {
    Node* current = node;
    while (current->left != NULL)
        current = current->left;
    return current;
}

// ---------------- DELETE ----------------
Node* deleteNode(Node* root, int id) {
    if (root == NULL)
        return root;

    if (id < root->id)
        root->left = deleteNode(root->left, id);
    else if (id > root->id)
        root->right = deleteNode(root->right, id);
    else {
        if (root->left == NULL || root->right == NULL) {
            Node *temp = root->left ? root->left : root->right;

            if (temp == NULL) {
                temp = root;
                root = NULL;
            } else
                *root = *temp;

            free(temp);
        } else {
            Node* temp = minValueNode(root->right);
            root->id = temp->id;
            strcpy(root->name, temp->name);
            root->age = temp->age;
            strcpy(root->disease, temp->disease);
            root->right = deleteNode(root->right, temp->id);
        }
    }

    if (root == NULL)
        return root;

    root->height = 1 + max(height(root->left), height(root->right));

    int balance = getBalance(root);

    // LL
    if (balance > 1 && getBalance(root->left) >= 0)
        return rightRotate(root);

    // LR
    if (balance > 1 && getBalance(root->left) < 0) {
        root->left = leftRotate(root->left);
        return rightRotate(root);
    }

    // RR
    if (balance < -1 && getBalance(root->right) <= 0)
        return leftRotate(root);

    // RL
    if (balance < -1 && getBalance(root->right) > 0) {
        root->right = rightRotate(root->right);
        return leftRotate(root);
    }

    return root;
}

// ---------------- DISPLAY ----------------
void inorder(Node *root) {
    if (root != NULL) {
        inorder(root->left);
        printf("\nID: %d | Name: %s | Age: %d | Disease: %s\n",
               root->id, root->name, root->age, root->disease);
        inorder(root->right);
    }
}

// ---------------- MAIN MENU ----------------
int main() {
    Node *root = NULL;
    int choice, id, age;
    char name[50], disease[50];

    while (1) {
        printf("\n==============================\n");
        printf(" PATIENT APPOINTMENT SYSTEM ");
        printf("\n==============================\n");
        printf("1. Add Patient\n");
        printf("2. Display Patients\n");
        printf("3. Search Patient\n");
        printf("4. Delete Patient\n");
        printf("5. Exit\n");
        printf("Enter choice: ");
        scanf("%d", &choice);

        switch (choice) {

        case 1:
            printf("Enter ID: ");
            scanf("%d", &id);
            printf("Enter Name: ");
            scanf("%s", name);
            printf("Enter Age: ");
            scanf("%d", &age);
            printf("Enter Disease: ");
            scanf("%s", disease);

            root = insert(root, id, name, age, disease);
            printf("✔ Patient added successfully!\n");
            break;

        case 2:
            printf("\n--- Patient List ---\n");
            inorder(root);
            break;

        case 3:
            printf("Enter ID to search: ");
            scanf("%d", &id);

            Node *res = search(root, id);
            if (res)
                printf("Found -> ID:%d Name:%s Age:%d Disease:%s\n",
                       res->id, res->name, res->age, res->disease);
            else
                printf("❌ Not Found\n");
            break;

        case 4:
            printf("Enter ID to delete: ");
            scanf("%d", &id);
            root = deleteNode(root, id);
            printf("✔ Deleted successfully (if existed)\n");
            break;

        case 5:
            printf("Exiting...\n");
            exit(0);

        default:
            printf("❌ Invalid choice\n");
        }
    }
}
