SERVLET = E’ una classe che riceve request e produce response. (versione base del controller)  
Lo troviamo con annotazioni di Java tipo @WebServlet (uguale al request mapping)

  
Il servlet è un concetto fondamentale in Java per la programmazione web. Secondo il contesto fornito

-Un servlet è essenzialmente una classe in Java.

-È progettato per ricevere richieste (request) e produrre risposte (response).

-È la versione base di un controller in Spring, ma è più limitato e verboso rispetto ai controller di Spring.

-I servlet estendono la classe HttpServlet, che è una classe base di Java.

-Possono essere annotati con @WebServlet per specificare l'URI a cui rispondono.

-Contengono metodi come doGet, doPost, doDelete, doPut che si attivano quando arriva una richiesta con il corrispondente verbo HTTP.

-Sono più limitati rispetto ai controller di Spring, ad esempio possono mappare solo un URI e non possono aggiungere facilmente altre funzionalità.

-In Spring, c'è una servlet speciale chiamata DispatcherServlet che riceve tutte le richieste e le indirizza ai controller appropriati.

Il servlet rappresenta quindi un concetto di base per la gestione delle richieste web in Java, su cui frameworks come Spring hanno poi costruito astrazioni più potenti e flessibili.

  
Nel nostro caso, Dispatcher Servlet , la servlet riceve TUTTE le request e le manda ai controller o rest controller corretti.