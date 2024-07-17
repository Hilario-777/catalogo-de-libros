<h1> Catalogo de libros </h1>
<h2>Hilario Salgado Antonio</h2>

Crear la base de datos MySQL
CREATE DATABASE BookCatalog;
USE BookCatalog;

CREATE TABLE books (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    author VARCHAR(255) NOT NULL,
    isbn VARCHAR(13) NOT NULL UNIQUE,
    published_date DATE,
    price DECIMAL(10, 2)
);
Configurar la base de datos en application.properties
Configura la conexión a la base de datos en el archivo src/main/resources/application.properties.
spring.datasource.url=jdbc:mysql://localhost:3306/BookCatalog
spring.datasource.username=root
spring.datasource.password=yourpassword
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true

Crear las clases de modelo, repositorio, servicio y controlador
// src/main/java/com/example/bookcatalog/model/Book.java
package com.example.bookcatalog.model;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import java.util.Date;

@Entity
public class Book {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;
    private String author;
    private String isbn;
    private Date publishedDate;
    private Double price;

    // Getters and Setters
}
Repositorio
// src/main/java/com/example/bookcatalog/repository/BookRepository.java
package com.example.bookcatalog.repository;

import com.example.bookcatalog.model.Book;
import org.springframework.data.jpa.repository.JpaRepository;

public interface BookRepository extends JpaRepository<Book, Long> {
}
Servicio
// src/main/java/com/example/bookcatalog/service/BookService.java
package com.example.bookcatalog.service;

import com.example.bookcatalog.model.Book;
import com.example.bookcatalog.repository.BookRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;

@Service
public class BookService {
    @Autowired
    private BookRepository bookRepository;

    public List<Book> getAllBooks() {
        return bookRepository.findAll();
    }

    public Optional<Book> getBookById(Long id) {
        return bookRepository.findById(id);
    }

    public Book saveBook(Book book) {
        return bookRepository.save(book);
    }

    public void deleteBook(Long id) {
        bookRepository.deleteById(id);
    }
}
Controlador
// src/main/java/com/example/bookcatalog/controller/BookController.java
package com.example.bookcatalog.controller;

import com.example.bookcatalog.model.Book;
import com.example.bookcatalog.service.BookService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;
import java.util.Optional;

@RestController
@RequestMapping("/api/books")
public class BookController {
    @Autowired
    private BookService bookService;

    @GetMapping
    public List<Book> getAllBooks() {
        return bookService.getAllBooks();
    }

    @GetMapping("/{id}")
    public ResponseEntity<Book> getBookById(@PathVariable Long id) {
        Optional<Book> book = bookService.getBookById(id);
        return book.map(ResponseEntity::ok)
                   .orElseGet(() -> ResponseEntity.notFound().build());
    }

    @PostMapping
    public Book createBook(@RequestBody Book book) {
        return bookService.saveBook(book);
    }

    @PutMapping("/{id}")
    public ResponseEntity<Book> updateBook(@PathVariable Long id, @RequestBody Book bookDetails) {
        Optional<Book> bookOptional = bookService.getBookById(id);
        if (!bookOptional.isPresent()) {
            return ResponseEntity.notFound().build();
        }
        Book book = bookOptional.get();
        book.setTitle(bookDetails.getTitle());
        book.setAuthor(bookDetails.getAuthor());
        book.setIsbn(bookDetails.getIsbn());
        book.setPublishedDate(bookDetails.getPublishedDate());
        book.setPrice(bookDetails.getPrice());
        Book updatedBook = bookService.saveBook(book);
        return ResponseEntity.ok(updatedBook);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteBook(@PathVariable Long id) {
        Optional<Book> bookOptional = bookService.getBookById(id);
        if (!bookOptional.isPresent()) {
            return ResponseEntity.notFound().build();
        }
        bookService.deleteBook(id);
        return ResponseEntity.ok().build();
    }
}
Ejecutar la aplicación
Ejecuta la clase principal de tu aplicación (BookCatalogApplication.java), y tu API de catálogo de libros debería estar disponible en http://localhost:8080/api/books.

Puedes probar la API con algunas peticiones como:

Obtener todos los libros:
curl -X GET http://localhost:8080/api/books
Crear un nuevo libro:
curl -X POST http://localhost:8080/api/books -H "Content-Type: application/json" -d '{"title":"Spring Boot in Action", "author":"Craig Walls", "isbn":"9781617292545", "publishedDate":"2016-01-01", "price":39.99}'
Actualizar un libro:
curl -X DELETE http://localhost:8080/api/books/1















