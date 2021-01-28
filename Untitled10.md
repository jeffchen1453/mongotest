

```python
from pymongo import MongoClient
```


```python
client = MongoClient(host='127.0.0.1', port = 27017)
```


```python
db = client['aiot']
collection = db['member']
collection
```




    Collection(Database(MongoClient(host=['127.0.0.1:27017'], document_class=dict, tz_aware=False, connect=True), 'aiot'), 'member')




```python
data_list = [
    {'name':'Chin-yu','phone':'0928778120','email':'chin_yu@gmail.com'},
    {'name':'Mimi','phone':'0904131313','email':'mimi@gmail.com'},
    {'name':'deeyo','phone':'0955944431','email':'deeyo@gmail.com'}
]
```


```python

result = collection.insert_many(data_list)
result
```




    <pymongo.results.InsertManyResult at 0x1e5b9e646c8>




```python
from flask import Flask
from flask_pymongo import PyMongo

app = Flask(__name__)
app.config["MONGO_URI"] = "mongodb://localhost:27017/myDatabase"
mongo = PyMongo(app)
```


```python
from flask import Flask,request
from flask_pymongo import PyMongo
from bson.objectid import ObjectId

```


```python
from flask import Flask,request,jsonify
from flask_pymongo import PyMongo
from bson.objectid import ObjectId

app = Flask(__name__)
app.config['MONGO_URI'] = "mongodb://localhost:27017/aiot"  #資料庫address
mongo = PyMongo(app)

#取得會員資料
@app.route("/members")
@app.route("/members/<id>",methods = ['GET'])
#使用 Flask 操作 MongoDB - 取得GET
def get_member(id = None):
    #如果沒有id，取得所有會員資料
    if id is None:
        members = mongo.db.member.find({})
        result = []
        for m in members:
            m['_id'] = str(m['_id'])
            result.append(m)
        return jsonify(result)
    else:
        result = mongo.db.member.find_one({'_id':ObjectId(id)})
        if result is not None:
            result['_id'] = str(result['_id']) #回傳id為ObjectId型別轉換str
        return jsonify(result)

#使用 Flask 操作 MongoDB - 新增POST一筆資料  
@app.route('/members', methods = ['POST'])
def add_member():
    name = request.form.get('name')
    phone = request.form.get('phone')
    email = request.form.get('email')
    result = mongo.db.member.insert_one({'name':name, 'phone':phone, 'email':email})
    return str(result.inserted_id)

#使用 Flask 操作 MongoDB - 刪除DELETE一筆資料
@app.route('/members/<id>', methods = ['DELETE'])
def remove_member(id):
    #先確認db內是否有資料
    m = mongo.db.member.find({'_id':ObjectId(id)})
    if m is not None:
        result = mongo.db.member.delete_one({'_id':ObjectId(id)})
    return 'The number of deleted data:' + str(result.deleted_count)

#使用 Flask 操作 MongoDB - 修改PUT一筆資料
@app.route("/members/<id>", methods = ['PUT'])
def update_member(id):
    result = 0
    #從form裡面取出資料
    name = request.form.get('name')
    phone = request.form.get('phone')
    email = request.form.get('email')
    #把新的資料用 dict 型別包裝
    new_data = {"$set":{"name":name, "phone":phone, "email":email}}
    #用update_one更新一筆資料，若要更新多筆可用update_many
    upd_result=mongo.db.member.update_one({"_id": ObjectId(id)}, new_data)
    if upd_result is not None:
        return 'The number of updated data:' + str(upd_result.modified_count)
    
if __name__ == '__main__':
    app.run()

```
