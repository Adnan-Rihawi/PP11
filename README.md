# PP11

## Goal

In this exercise you will:

* Design and implement a **doubly linked list** as a reusable library.
* Create and interact with a **singly linked list** via an interactive menu, using dynamic memory (`malloc`/`free`).
* Parse a JSON array from a file using **jansson**, store objects in a linked list, and manipulate them in C.

**Important:** Start a stopwatch when you begin and work uninterruptedly for **90 minutes**. When time is up, stop immediately and record where you paused.

---

## Workflow

1. **Fork** this repository on GitHub.
2. **Clone** your fork locally.
3. Create a `solutions/` directory at the project root:

   ```bash
   mkdir solutions
   ```
4. For each task, add the specified source files under `solutions/`.
5. **Commit** and **push** your changes to GitHub.
6. **Submit** your GitHub repo link for review.

---

## Prerequisites

* GNU C compiler (`gcc`).
* Understanding of pointers and dynamic memory (`malloc`, `free`).
* Jansson library installed (`libjansson-dev`).
* Linker flags (`-ljansson`).

---

## Tasks

### Task 1: Compile-Time Doubly Linked List

**Objective:** Implement and traverse a doubly linked list entirely with compile-time allocation.

1. Create `solutions/dlist_static.c` and include `dlist.h` (which defines `DNode` and `DList`).
2. Define a static array of five `DNode` elements:

   ```c
   #include <stdio.h>
   #include "dlist.h"  // defines DNode and DList

   // Static nodes for values 1–5
   DNode nodes[5];
   DList list;

   int main(void) {
       // Initialize static nodes
       for (int i = 0; i < 5; i++) {
           // Example: store node index as data
           nodes[i].data = (void*)(long)(i + 1);
           nodes[i].prev = (i > 0) ? &nodes[i - 1] : NULL;
           nodes[i].next = (i < 4) ? &nodes[i + 1] : NULL;
       }
       list.head = &nodes[0];
       list.tail = &nodes[4];

       // Traverse forward
       printf("Forward: ");
       for (DNode *p = list.head; p != NULL; p = p->next) {
           printf("%ld ", (long)p->data);
       }
       printf("\n");

       // Traverse backward
       printf("Backward: ");
       for (DNode *p = list.tail; p != NULL; p = p->prev) {
           printf("%ld ", (long)p->data);
       }
       printf("\n");

       return 0;
   }
   ```
3. Compile and run:

   ```bash
   gcc -o solutions/dlist_static solutions/dlist_static.c
   ./solutions/dlist_static
   ```

#### Reflection Questions

1. **How do you link `prev` and `next` pointers correctly using a static array?**
   Man nutzt einfach die festen Positionen im Array: nodeslil prev zeigt auf nodes[i-1], und nodes(i]. next zeigt auf nodes[i+1]. Für das erste Element ist prev = NULL, für das letzte next = NULL.
So entsteht die doppelt verkettete Liste komplett ohne malloc.
3. **What are advantages and limitations of compile-time vs. dynamic allocation?**
   Vorteile: Kein dynamischer Speicher → keine Leaks, keine Fehler durch malloc Sehr schnell, da alles im statischen Speicher liegt Einfacher Code, ideal für feste kleine Listen
Nachteile und limits: Größe ist fix und kann nicht wachsen Speicher wird immer reserviert, auch wenn die Liste nicht genutzt wird Weniger flexibel als dynamische Datenstrukturen
5. **How would you extend this static list to include additional data fields?**
   Man ergänzt einfach die DNode-Struktur um neue Felder, die statischen Array-Elemente enthalten dann automatisch die neuen Felder und können wie gewohnt initialisiert werden.

---

### Task 2: Interactive Singly Linked List

**Objective:** Build a menu-driven program to manipulate a singly linked list at runtime.

1. Define `SNode` and function prototypes in `solutions/slist.h`:

   ```c
   typedef struct SNode {
       int value;
       struct SNode *next;
   } SNode;

   SNode* add_node(SNode *head, int value);
   int    count_nodes(SNode *head);
   void   free_list(SNode *head);
   ```
2. Implement `add_node`, `count_nodes`, and `free_list` in `solutions/slist.c` using `malloc` and `free`.
3. Create `solutions/slist_main.c`:

   ```c
   #include <stdio.h>
   #include "slist.h"

   int main(void) {
       SNode *head = NULL;
       int choice, val;

       do {
           printf("1) Count nodes\n2) Add node\n3) Exit\n> ");
           if (scanf("%d", &choice) != 1) break;
           switch (choice) {
               case 1:
                   printf("Count: %d\n", count_nodes(head));
                   for (SNode *p = head; p; p = p->next)
                       printf("Node at %p: %d\n", (void*)p, p->value);
                   break;
               case 2:
                   printf("Value: ");
                   scanf("%d", &val);
                   head = add_node(head, val);
                   break;
               case 3:
                   free_list(head);
                   break;
               default:
                   printf("Invalid option\n");
           }
       } while (choice != 3);

       return 0;
   }
   ```
4. Compile and run:

   ```bash
   gcc -c solutions/slist.c -o solutions/slist.o
   gcc -o solutions/slist_main solutions/slist_main.c solutions/slist.o
   ./solutions/slist_main
   ```

#### Reflection Questions

1. **Why is `malloc` necessary when adding nodes dynamically?** Weil die Liste zur Laufzeit wächst und jeder neue Node Speicher im Heap braucht. Ohne malloc gäbe es keinen Platz für neue Elemente, da die Größe nicht vorher feststeht.
2. **How can you traverse the list to print each node’s address and value?** Man läuft einfach mit einem Pointer durch die Liste, man startet bei head und folgt jedem next-Pointer bis NULL.
3. **What are the consequences of not freeing the list before exit?** Der reservierte Heap-Speicher bleibt belegt
- Memory Leak. Das Programm beendet sich zwar, aber es hat Speicher hinterlassen, der nicht sauber freigegeben wurde. Bei langen Programmen oder vielen Operationen kann das zu ernsthaften Problemen fuhren.

---

### Task 3: JSON Array to Linked List

**Objective:** Parse a JSON array from a file into a linked list of C structs, using `getopt` to accept the input filename.

1. Create `solutions/json_list.h`:

   ```c
   #include <jansson.h>
   typedef struct JObject {
       char *name;
       int age;
       struct JObject *next;
   } JObject;

   JObject* parse_json(const char *filename);
   void     print_list(JObject *head);
   void     free_list(JObject *head);
   ```

2. Implement `solutions/json_list.c`:

   * Use `json_load_file` to load an array of objects with fields `"name"` and `"age"`.
   * For each element, allocate a `JObject`, duplicate the `name`, store `age`, and link into a list.

3. Create `solutions/json_main.c` to use `getopt`:

   ```c
   #include "json_list.h"
   #include <stdio.h>
   #include <stdlib.h>
   #include <unistd.h>

   void usage(const char *prog) {
       fprintf(stderr, "Usage: %s -i <file.json>
   ```

", prog);
exit(EXIT\_FAILURE);
}

int main(int argc, char \*argv\[]) {
int opt;
char \*filename = NULL;

```
   while ((opt = getopt(argc, argv, "i:")) != -1) {
       if (opt == 'i') {
           filename = optarg;
       } else {
           usage(argv[0]);
       }
   }
   if (!filename) {
       usage(argv[0]);
   }

   JObject *head = parse_json(filename);
   print_list(head);
   free_list(head);
   return 0;
```

}

````
4. Compile and run:
```bash
gcc -c solutions/json_list.c -o solutions/json_list.o -ljansson
gcc -o solutions/json_main solutions/json_main.c solutions/json_list.o -ljansson
./solutions/json_main -i data.json
````

#### Suggested Student Modifications

1. **Add** `void remove_by_age(JObject **head, int age)` to remove matching nodes.
2. **Extend** `JObject` with a new field (e.g., `char *email`) and update parsing accordingly.

#### Reflection Questions

1. **How does using `getopt` make the program more flexible than `argv[1]`?** Mit getopt kannst du benannte Optionen wie -i verwenden, statt dich auf eine feste Position in argv[] zu verlassen. Das erlaubt mehr Argumente, beliebige
Reihenfolge und klare Fehlermeldungen.
2. **What happens if the user omits the `-i` option?** Dann bleibt filename ==
NULL und das Programm ruft usage()
auf - Fehlerhinweis +
Programmabbruch. So wird verhindert, dass ohne gültige Eingabedatei
weitergearbeitet wird.
3. **How can you validate that the JSON file loaded is indeed an array?**
Um zu prüfen, ob die geladene JSON-Datei wirklich ein Array ist, verwende ich json_is_array(root). Nur wenn diese Funktion true zurückgbt, weiß ich sicher, dass die Struktur ein Array ist und ich es gefahrlos iterieren kann.
Andernfalls breche ich ab und melde einen Fehler.
---

**Remember:** Stop after **90 minutes** and record where you stopped.
