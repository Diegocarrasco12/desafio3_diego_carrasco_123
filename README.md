# desafio3_diego_carrasco_123
-- Active: 1729897964143@@127.0.0.1@5432@desafio3_diego_carrasco_123
Database Client: Add New Connection

CREATE TABLE usuarios (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    nombre VARCHAR(100),
    apellido VARCHAR(100),
    rol VARCHAR(50)
);

SELECT * FROM usuarios;

INSERT INTO usuarios (id, email, nombre, apellido, rol) VALUES
    (1, 'usuario1@example.com', 'Antonio', 'Gutierrez', 'usuario'),
    (2, 'usuario2@example.com', 'Roberto', 'Fernández', 'administrador'),
    (3, 'usuario3@example.com', 'Diego', 'Carrasco', 'usuario'),
    (4, 'usuario4@example.com', 'Belen', 'Angulo', 'administrador'),
    (5, 'usuario5@example.com', 'Sandro', 'Reyes', 'usuario');

SELECT * FROM usuarios;

CREATE TABLE Post (
    id SERIAL PRIMARY KEY,
    titulo VARCHAR(255) NOT NULL,
    contenido TEXT,
    fecha_creacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    fecha_actualizacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    destacado BOOLEAN DEFAULT FALSE,
    usuario_id BIGINT,
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id) ON DELETE SET NULL
);

SELECT * FROM post;

INSERT INTO Post (id, titulo, contenido, usuario_id, destacado) VALUES
    (1, 'Post 1', 'Contenido del primer post', 1, TRUE),
    (2, 'Post 2', 'Contenido del segundo post', 2, FALSE),
    (3, 'Post 3', 'Contenido del primer post de usuario', 3, TRUE),
    (4, 'Post 4', 'Contenido del segundo post de usuario', 4, FALSE),
    (5, 'Post 5', 'Este post no tiene un usuario asignado', NULL, FALSE);

SELECT * FROM Post;

CREATE TABLE Comentarios (
    id SERIAL PRIMARY KEY,
    contenido TEXT NOT NULL,
    fecha_creacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    usuario_id BIGINT,
    post_id BIGINT,
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id) ON DELETE SET NULL,
    FOREIGN KEY (post_id) REFERENCES Post(id) ON DELETE CASCADE
);

SELECT * FROM comentarios;

INSERT INTO Comentarios (id, contenido, usuario_id, post_id) VALUES
    (1, 'Primer comentario en el post 1', 1, 1),
    (2, 'Segundo comentario en el post 1', 2, 1),
    (3, 'Tercer comentario en el post 1', 3, 1),
    (4, 'Primer comentario en el post 2', 1, 2),
    (5, 'Segundo comentario en el post 2', 2, 2);

SELECT * FROM comentarios;


--  2. Cruza los datos de la tabla usuarios y posts, mostrando las siguientes columnas:  nombre y email del usuario junto al título y contenido del post.  (1 Punto)
SELECT 
    usuarios.nombre,
    usuarios.email,
    Post.titulo,
    Post.contenido
FROM 
    usuarios
JOIN 
    Post ON usuarios.id = Post.usuario_id;


-- 3. Muestra el id, título y contenido de los posts de los administradores. a. El administrador puede ser cualquier id.  (1 Punto).
SELECT 
    Post.id,
    Post.titulo,
    Post.contenido
FROM 
    Post
JOIN 
    usuarios ON Post.usuario_id = usuarios.id
WHERE 
    usuarios.rol = 'administrador';

-- 4. Cuenta la cantidad de posts de cada usuario.  a. La tabla resultante debe mostrar el id e email del usuario junto con la cantidad de posts de cada usuario. (1 Punto)
SELECT 
    usuarios.id,
    usuarios.email,
    COUNT(Post.id) AS cantidad_de_posts
FROM 
    usuarios
LEFT JOIN 
    Post ON usuarios.id = Post.usuario_id
GROUP BY 
    usuarios.id, usuarios.email;

-- 5. Muestra el email del usuario que ha creado más posts. Aquí la tabla resultante tiene un único registro y muestra solo el email.(1 Punto)
SELECT 
    usuarios.email
FROM 
    usuarios
JOIN 
    Post ON usuarios.id = Post.usuario_id
GROUP BY 
    usuarios.id, usuarios.email
ORDER BY 
    COUNT(Post.id) DESC
LIMIT 1;

--6. Muestra la fecha del último post de cada usuario.(1 Punto)
SELECT 
    usuarios.id,
    usuarios.email,
    MAX(Post.fecha_creacion) AS fecha_ultimo_post
FROM 
    usuarios
LEFT JOIN 
    Post ON usuarios.id = Post.usuario_id
GROUP BY 
    usuarios.id, usuarios.email;

-- 7. Muestra el título y contenido del post (artículo) con más comentarios.(1 Punto)
SELECT 
    Post.titulo,
    Post.contenido
FROM 
    Post
JOIN 
    Comentarios ON Post.id = Comentarios.post_id
GROUP BY 
    Post.id, Post.titulo, Post.contenido
ORDER BY 
    COUNT(Comentarios.id) DESC
LIMIT 1;

-- 8. Muestra en una tabla el título de cada post, el contenido de cada post y el contenido de cada comentario asociado a los posts mostrados, junto con el email del usuario que lo escribió.(1 Punto)
SELECT 
    Post.titulo,
    Post.contenido AS contenido_post,
    Comentarios.contenido AS contenido_comentario,
    usuarios.email
FROM 
    Post
LEFT JOIN 
    Comentarios ON Post.id = Comentarios.post_id
LEFT JOIN 
    usuarios ON Comentarios.usuario_id = usuarios.id;

-- 9. Muestra el contenido del último comentario de cada usuario.(1 Punto)
SELECT 
    usuarios.id,
    usuarios.email,
    Comentarios.contenido
FROM 
    Comentarios
JOIN 
    usuarios ON Comentarios.usuario_id = usuarios.id
WHERE 
    Comentarios.fecha_creacion = (
        SELECT MAX(fecha_creacion)
        FROM Comentarios AS c
        WHERE c.usuario_id = Comentarios.usuario_id
    );

-- 10. Muestra los emails de los usuarios que no han escrito ningún comentario.(1 Punto)
SELECT 
    usuarios.email
FROM 
    usuarios
LEFT JOIN 
    Comentarios ON usuarios.id = Comentarios.usuario_id
WHERE 
    Comentarios.id IS NULL;
