---
topic: "REST API Design — pattern per API web"
parent: "[[BE-NOTES/Java/Spring/Web/Web|Web]]"
nav_prev: "[[Controller e REST.md]]"
nav_next: "[[DTO e Mappers.md]]"
---


Pattern e buone pratiche per la progettazione di API RESTful, come implementate in [[TaskMngr]].

## Argomenti

- [[BE-NOTES/Java/Spring/Web/ApiResponse Pattern|ApiResponse Pattern]] — envelope standard per risposte API (successo/errore/dati)
- [[BE-NOTES/Java/Spring/Web/Global Exception Handler|Global Exception Handler]] — @RestControllerAdvice, gestione centralizzata errori
- [[BE-NOTES/Java/Spring/Web/Validazione con Bean Validation|Validazione con Bean Validation]] — @Valid, @NotBlank, custom validators
- [[BE-NOTES/Java/Spring/Web/Paginazione|Paginazione]] — Page/Pageable, Spring Data Sorting, slice
