# Imagen base
FROM python:3.11-slim

# Crear carpeta
WORKDIR /app

# Copiar dependencias e instalarlas
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copiar el resto de la app
COPY . .

# Ejecutar script para crear la base de datos
RUN python create_db.py

# Exponer puerto
EXPOSE 5000

# Ejecutar la app Flask
CMD ["python", "vulnerable_flask_app.py"]
