# Phone book

The program `phone_book.c` is a partially completed phone book
program. It is a command line program that will maintain a database of
phone numbers and names in a file `directory.db`. 

It can be used in the following way. 

1. `./phone_book add name number`
   This command is used to add the `name` and `number` to the
   database. If the database file (
   
2. `./phone_book list`
   This command will list all the users and numbers in the system
   
3. `./phone_book search name`
   This command will search for the entry with name `name` and print
   the phone number if `name` exists. Otherwise it will print `no
   match` and exit with a non-zero status.
   
4. `./phone_book delete name`
   This command will search for the entry with name `name` and delete
   it from the database if `name` exists. Otherwise it will print `no
   match` and exit with a non-zero status.

# Design notes

The program loads the entries in the database into a linked list. Each
node of the list contains the name and phone number. 

The program has decent error checking. It will abort if you try to
list when the database file is non existent. It will also print proper
error messages if you type a wrong or malformed command. 

The program allocates memory to create the linked list. It should be
fully freed before the program exits. `valgrind` should show no leaks possible.

# Example usage

A pre compiled version of the program with all the features mentioned
below is available in the repository as `good_phone_book`. You can use
this to compare the output of your program.

## Error handling

    %  ls directory.db
    /bin/ls: cannot access 'directory.db': No such file or directory                                              
    % ./phone_book list
    Couldn't open database file: No such file or directory
    % ./phone_book search john
    Couldn't open database file: No such file or directory
    % ./phone_book delete john
    Couldn't open database file: No such file or directory

## Adding and listing

    % ./phone_book add john 1234567890
    % ./phone_book list
    john                 : 1234567890
    Total entries :  1
    % ./phone_book add james 0987654321
    % ./phone_book list
    john                 : 1234567890
    james                : 0987654321
    Total entries :  2

## Searching

    % ./phone_book search mary
    no match
    % ./phone_book search james
    0987654321

## Deleting

    % ./phone_book add jack 5432167890
    % ./phone_book delete mary
    no match
    % ./phone_book list
    john                 : 1234567890
    james                : 0987654321
    jack                 : 5432167890
    Total entries :  3
    % ./phone_book delete james
    % ./phone_book list
    john                 : 1234567890
    jack                 : 5432167890
    Total entries :  2
    % ./phone_book delete john
    % ./phone_book list
    jack                 : 5432167890
    Total entries :  1

# Tasks

There is a comment "TBD" in various parts of the phone_book.c file to
indicate what parts of the code need to be implemented.

1. The `total entries` is not currently printed by the `list`
   command. This needs to be fixed. 
1. The delete functionality has to be implemented similar to the others
   (including error checking etc.). The command line parsing works but
   it doesn't actually do any deletion. Make sure that all files are
   closed and all memory is freed.
1. The search functionality is not implemented. The command is also
   not recognised. This needs to be fixed. Make sure that all files are
   closed and all memory is freed.
1. The `free_entries` function should walk through the linked list and
   free all the nodes in it. It's not implemented. You can check to
   see if it's properly implemented and running 
   
         valgrind ./phone_book search jack
         
   The output of `valgrind` should have a line like this. This will
   happen only if `free_entries` is properly implemented and all
   memory is freed.
   
           All heap blocks were freed -- no leaks are possible

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

const char *DB="directory.db";

struct entry0 {
  char name[20];
  char phone[20];
  struct entry0 *next;
};

typedef struct entry0 entry;

/* Command handlers */
void add(char *, char *);
int search(FILE *,char *);
void list(FILE *);
int delete(FILE *, char *);

/* Utility functions  */
FILE * open_db_file(); /* Opens the database file. Prints error and
                          quits if it's not available */
void print_usage(char *, char *);  /* Prints usage */
entry *load_entries(FILE *);         /* Load all entries from the
                                      database file. Returns pointer
                                      to first entry */
entry *create_entry_node(char *, char *);  /* Create a new entry
                                              node. Has to be freed by
                                              user. */
void free_entries(entry *); /* TBD Given the first node of a linked list
                               of entries, will free all the nodes */ 

void write_all_entries(entry *); /* Given the first node of a linked
                                    list of entries, will delete the
                                    database file on the disk and save
                                    the given entries into the file */


int main(int argc, char *argv[]) {
  if (argc == 1) {
    print_usage("Insufficient arguments", argv[0]);
    exit(1);
  } 

  if (strcmp(argv[1], "add") == 0) {   /* Handle add */
    if (argc != 4) {
      print_usage("Improper arguments for add", argv[0]);
      exit(1);
    }
    char *name = argv[2];
    char *phone = argv[3];
    add(name, phone);
    exit(0);
  } else if (strcmp(argv[1], "list") == 0) {  /* Handle list */
    if (argc != 2) {
      print_usage("Improper arguments for list", argv[0]);
      exit(1);
    }
    FILE *fp = open_db_file();
    list(fp);
    fclose(fp);
    exit(0);
  } else if (strcmp(argv[1], "search") == 0) {  /* Handle search */
    if(argc != 3){
       print_usage("Improper arguments for search", argv[0]);
      exit(1);
    }/* TBD  */
    FILE *fp = open_db_file();
    char *name = argv[2];
    if (!search(fp, name)) {
      printf("no match\n");
      fclose(fp);
      exit(1);
    }
    fclose(fp);
    exit(0); 
  } else if (strcmp(argv[1], "delete") == 0) {  /* Handle delete */
    if (argc != 3) {
      print_usage("Improper arguments for delete", argv[0]);
      exit(1);
    }
    FILE *fp = open_db_file();
    char *name = argv[2];
    if (!delete(fp, name)) {
      printf("no match\n");
      fclose(fp);
      exit(1);
    }
    fclose(fp);
    exit(0);
  } else {
    print_usage("Invalid command", argv[0]);
    exit(1);
  }
}

FILE *open_db_file() {
  FILE *fp=fopen(DB, "r");
  if (fp == NULL) {
    perror("Couldn't open database file");
    exit(1);
  }
  return fp;
}
  
void free_entries(entry *p) {
  /* TBD */
 while(p!=NULL){
  free(p);
  p =p->next;
 }
  
  //printf("Memory is not being freed. This needs to be fixed!\n");  
}

void print_usage(char *message, char *progname) {
  printf("Error : %s\n", message);
  printf("Usage: %s command arguments\n", progname);
  printf("\nAvailable commands: \n-------------------\n");
  printf("add name phonenumber\n");
  printf("    Used to add a new name to the database.\n    name and phone number should not have any\n    spaces in them\n");
  printf("search name\n");
  printf("    Prints the phone number of the given name\n    if it's there in the database. Prints\n    'no match' if not found\n");
  printf("list\n");
  printf("    Lists all names and phone numbers in the\n    system along with a total count at the end.\n");
  printf("delete name\n");
  printf("    Deletes the entry for the name in the database.\n    Prints 'no match' if there's no such name.\n");
}

entry *
create_entry_node(char *name, char *phone) {
  entry *ret;
  ret = malloc(sizeof(entry));
  strcpy(ret->name, name);
  strcpy(ret->phone, phone);
  ret->next = NULL;
  return ret;
}

entry *
load_entries(FILE *fp) {
  char name[20], phone[20];
  memset(name, '\0', 20);
  memset(phone, '\0', 20);
  entry *ret = NULL;
  entry *current = NULL;
  entry *tmp = NULL;
  /* Description of %20[^,\n]
     % is the start of the specifier (like %s, %i etc.)
     20 is the maximum number of characters that this will take. We
        know that names and phone numbers will be 20 bytes maximum so
        we limit it to that. %20s will read in 20 character strings
        (including the , to separate the name and phone number. That's
        why we use
    [^,\n] Square brackets are used to indicate a set of allowed
           characters [abc] means only a, b, or c. With the ^, it's
           used to specify a set of disallowed characters. So [^abc]
           means any character *except* a, b, or c. [^,] means any
           character except a , [^,\n] means any character except a
           comma(,) or a newline(\n).
    %20[^,\n] will match a string of characters with a maximum length
     of 20 characters that doesn't have a comma(,) or a newline(\n).
  */        
  while (fscanf(fp, "%20[^,\n],%20[^,\n]\n", name, phone) != EOF) {
    tmp = create_entry_node(name, phone);
    if (ret == NULL)
      ret = tmp;
    else
      current->next = tmp;
    current = tmp;
  }
  return ret;
}

void write_all_entries(entry * p) {
  FILE *fp = fopen(DB, "w");
  while (p != NULL) {
    fprintf(fp, "%s,%s\n", p->name, p->phone);
    p = p->next;
  }
  fclose(fp);
}


void add(char *name, char *phone) {
  FILE *fp = fopen(DB, "a");
  fprintf(fp, "%s,%s\n", name, phone);
  fclose(fp);
}
int search(FILE *db_file,char *name){
  entry *p = load_entries(db_file);
  entry *base = p;
  int found = 0;
  while(p!=NULL){
  if(strcmp(p->name,name) == 0){
  printf("%s\n",p->phone);
 
  found = 1;
  }
   p=p->next;
  }
  free_entries(base);
  return found;
}

int size = 0;
void list(FILE *db_file) {
  entry *p = load_entries(db_file);
  entry *base = p;
  int count = 0;
  while (p!=NULL) {
    printf("%-20s : %10s\n", p->name, p->phone);
    count++;
    p=p->next;
  }
  /* TBD print total count */
  size =count;
  printf("Total entries :  %d\n",count);
  free_entries(base);
 
}


int delete(FILE *db_file, char *name) {
  entry *p = load_entries(db_file);
  entry *base = p;
  entry *prev = NULL;
  entry *del = NULL ; /* Node to be deleted */
  int deleted = 0;

  while (p!=NULL) {
    if (strcmp(p->name, name) == 0) {
      /* Matching node found. Delete it from the linked list.
         Deletion from a linked list like this
   
             p0 -> p1 -> p2
         
         means we have to make p0->next point directly to p2. The p1
         "node" is removed and free'd.
         
         If the node to be deleted is p0, it's a special case. 
      */

      /* TBD */
      
       if(strcmp(base->name, name) == 0){
         base = p->next;
       }
      else{ p = p->next;
      
       prev->next = p;
          }
      
       deleted = 1;
       break;
    }
    else{
      prev = p;
      p = p->next;
    }
   
  }
  write_all_entries(base);
  free_entries(base);
  return deleted;
}
