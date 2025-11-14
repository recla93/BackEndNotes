### 1. Generics  
  
#java/generics #data-structures  
  
> [!info] Definizione  
> I **Generics** permettono di parametrizzare i tipi, scrivendo codice che funziona con qualsiasi tipo specificato in fase di compilazione.  
  
### Vantaggi Principali  
  
- ==Type Safety==: errori individuati in compilazione  
- ==Eliminazione cast==: no casting esplicito necessario  
- ==Riutilizzo codice==: una classe gestisce più tipi  
  
### Sintassi Base  
  
```java  
// Classe generica semplice  
public class Box<T> {  
    private T content;  
      
    public void set(T content) {  
        this.content = content;  
    }  
      
    public T get() {  
        return content;  
    }  
}  
  
// Utilizzo  
Box<String> stringBox = new Box<>();  
stringBox.set("Hello");  
String value = stringBox.get();  
```  
  
### Binary Search Tree Generico  
  
> [!example] Esempio: BST con Generics  
> Implementazione completa di un albero binario di ricerca generico  
  
```java  
public class BinarySearchTree<T extends Comparable<T>> {  
    private class Node {  
        T data;  
        Node left;  
        Node right;  
          
        Node(T data) {  
            this.data = data;  
            this.left = null;  
            this.right = null;  
        }  
    }  
      
    private Node root;  
      
    public void insert(T value) {  
        root = insertRec(root, value);  
    }  
      
    private Node insertRec(Node node, T value) {  
        if (node == null) {  
            return new Node(value);  
        }  
          
        if (value.compareTo(node.data) < 0) {  
            node.left = insertRec(node.left, value);  
        } else if (value.compareTo(node.data) > 0) {  
            node.right = insertRec(node.right, value);  
        }  
          
        return node;  
    }  
      
    public boolean search(T value) {  
        return searchRec(root, value);  
    }  
      
    private boolean searchRec(Node node, T value) {  
        if (node == null) return false;  
        if (value.equals(node.data)) return true;  
          
        return value.compareTo(node.data) < 0   
            ? searchRec(node.left, value)   
            : searchRec(node.right, value);  
    }  
}  
```  
  
### Vincoli sui Generics  
  
```java  
// Vincolo con extends  
public class NumberBox<T extends Number> {  
    private T value;  
      
    public double getDoubleValue() {  
        return value.doubleValue();  
    }  
}  
  
// Interfacce multiple  
public class Repository<E extends BaseEntity & Serializable> {  
    // E deve implementare entrambe  
}  
```  
  
> [!tip] Quando utilizzarli  
> - Collezioni personalizzate (Liste, Set, Map, Alberi)  
> - Repository e DAO per diverse entità  
> - Algoritmi riutilizzabili  
> - Builder pattern con parametri variabili  
  
***  
  