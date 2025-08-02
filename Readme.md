¡Perfecto! Si solo quieres que `app.py` actúe como **una API backend (CRUD)** con Flask conectada a **MongoDB Atlas**, sin renderizar templates HTML (solo JSON responses), el archivo sería algo así como:

---

### ✅ `app.py` (CRUD backend puro con Flask + MongoDB)

```python
from flask import Flask, request, jsonify
from flask_cors import CORS
from pymongo import MongoClient
from bson.objectid import ObjectId
from dotenv import load_dotenv
import os

load_dotenv()

app = Flask(__name__)
CORS(app)  # Para permitir peticiones desde frontend

MONGO_URI = os.getenv("MONGO_URI")
client = MongoClient(MONGO_URI)
db = client["mydatabase"]
collection = db["items"]

# Get all items
@app.route("/items", methods=["GET"])
def get_items():
    items = list(collection.find())
    for item in items:
        item["_id"] = str(item["_id"])
    return jsonify(items)

# Get single item by ID
@app.route("/items/<id>", methods=["GET"])
def get_item(id):
    item = collection.find_one({"_id": ObjectId(id)})
    if item:
        item["_id"] = str(item["_id"])
        return jsonify(item)
    return jsonify({"error": "Item not found"}), 404

# Create new item
@app.route("/items", methods=["POST"])
def create_item():
    data = request.json
    result = collection.insert_one(data)
    return jsonify({"_id": str(result.inserted_id)}), 201

# Update item by ID
@app.route("/items/<id>", methods=["PUT"])
def update_item(id):
    data = request.json
    result = collection.update_one({"_id": ObjectId(id)}, {"$set": data})
    if result.matched_count:
        return jsonify({"message": "Item updated"})
    return jsonify({"error": "Item not found"}), 404

# Delete item by ID
@app.route("/items/<id>", methods=["DELETE"])
def delete_item(id):
    result = collection.delete_one({"_id": ObjectId(id)})
    if result.deleted_count:
        return jsonify({"message": "Item deleted"})
    return jsonify({"error": "Item not found"}), 404

if __name__ == "__main__":
    app.run(debug=True)
```

---

### 📦 Estructura básica del proyecto

```
/mi-app-backend
│
├── app.py
├── .env
├── requirements.txt
└── README.md
```

### 🧪 `.env` (no lo subas a GitHub)

```
MONGO_URI=mongodb+srv://<usuario>:<password>@<cluster>.mongodb.net/?retryWrites=true&w=majority
```

### 📦 `requirements.txt`

```
Flask
flask-cors
pymongo
python-dotenv
```

---

### 🚀 Cómo ejecutar

```bash
pip install -r requirements.txt
python app.py
```

---

### 🌐 Ejemplo de endpoints

* `GET     /items` → Todos los items
* `GET     /items/<id>` → Un solo item
* `POST    /items` → Crear item (JSON body)
* `PUT     /items/<id>` → Actualizar item
* `DELETE  /items/<id>` → Eliminar item

---
