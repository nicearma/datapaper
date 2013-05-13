# DataPaper
This is configuration for an data base NoSQL with CouchDB V-1.2.
##How to configure.
Before continue you had to have installed CouchDB in your server.
* Make a user
* Make one database, on this case datapaper (you can change this name)
* Make un desing web with the name public and create 3 type of view and one validation function
  *The first view by_all2
<pre><code>
function(doc) {
  emit(doc._id, doc);
}
</code></pre>
  * The second view by_all
<pre><code>
function(doc) {
for(var key in doc.public)
  emit(doc._id, doc.public[key]);
}
</code></pre>
	* The third view by_type
<pre><code>
function(doc) {
var infoUser=new RegExp("user");
for(var key in doc.public){
	if(infoUser.test(doc.public[key].type)){
	emit([doc._id,'user-information'], doc.public[key]);
	}else{
emit([doc._id,'document'], doc.public[key]);
	}
  }
}	
</code></pre>
	* And the validation function
<pre><code>
function(newDoc, oldDoc, userCtx, secObj) {
    if (newDoc._deleted === true) {
        if ((userCtx.roles.indexOf('_admin') !== -1) ||
                (userCtx.name == oldDoc.name)) {
            return;
        } else {
            throw({forbidden: 'Only admins may delete other user docs.'});
        }
    }
    if (!newDoc._id) {
        throw({forbidden: 'doc.name is required'});
    }
	if(!secObj){
        throw({forbidden: 'You dont have the session for make a change.'});
    }
 	if (!newDoc.private) {
        throw({forbidden: 'you dont have private, and is required'});
    } else {
    	//for put more type of document please change this objet
		if(newDoc.hasOwnProperty("edit_by")){
		var documentPrivate = "name, url";
        for (var key in newDoc.private) {
            if (!(documentPrivate.indexOf(key) > -1)) {
                throw({forbidden: key + ' private is required'});
            }
		 }
      	}
    }
    if (!newDoc.public) {

        throw({forbidden: 'you dont have public, and is required'});
    } else {
        for (j=0; j < newDoc.public.length; j++) {
            
            for (i = 0; i < newDoc.public[j].length; i++) {
                if (!newDoc.public[key][i].description) {
                    throw({forbidden: 'in ' + key + 'you dont have description, and  is required'});
                }
                if (!newDoc.public[key][i].url) {
                    throw({forbidden: 'in ' + key + 'you dont have url, and  is required'});
                }
				if (!newDoc.public[key][i].type) {
                    throw({forbidden: 'in ' + key + 'you dont have url, and  is required'});
                }
            }

        }
    }
    if (newDoc.roles && !isArray(newDoc.roles)) {
        throw({forbidden: 'doc.roles must be an array'});
    }
}
</code></pre>
#### Here is the all document.
<pre><code>
{
   "_id": "_design/public",
   "language": "javascript",
   "views": {
       "by_all2": {
           "map": "function(doc) {\n  emit(doc._id, doc);\n}"
       },
       "by_all": {
           "map": "function(doc) {\nfor(var key in doc.public)\n  emit(doc._id, doc.public[key]);\n}"
       },
       "by_type": {
           "map": "function(doc) {\nvar infoUser=new RegExp(\"user\");\nfor(var key in doc.public){\n\tif(infoUser.test(doc.public[key].type)){\n\temit([doc._id,'user-information'], doc.public[key]);\n\t}else{\nemit([doc._id,'document'], doc.public[key]);\n\t}\n  }\n}\t"
       }
   },
   "validate_doc_update": "function(newDoc, oldDoc, userCtx, secObj) {\n    if (newDoc._deleted === true) {\n        if ((userCtx.roles.indexOf('_admin') !== -1) ||\n                (userCtx.name == oldDoc.name)) {\n            return;\n        } else {\n            throw({forbidden: 'Only admins may delete other user docs.'});\n        }\n    }\n    if (!newDoc._id) {\n        throw({forbidden: 'doc.name is required'});\n    }\nif(!secObj){\n        throw({forbidden: 'You dont have the session for make a change.'});\n    }\n if (!newDoc.private) {\n        throw({forbidden: 'you dont have private, and is required'});\n    } else {\n    //for put more type of document please change this objet\n\tif(newDoc.hasOwnProperty(\"edit_by\")){\n\tvar documentPrivate = \"name, url\";\n        for (var key in newDoc.private) {\n            if (!(documentPrivate.indexOf(key) > -1)) {\n                throw({forbidden: key + ' private is required'});\n            }\n\t}\n       \n          \n        }\n\n    }\n   \n    if (!newDoc.public) {\n\n        throw({forbidden: 'you dont have public, and is required'});\n    } else {\n//for put more type of document please change this objet\n   \n        for (j=0; j < newDoc.public.length; j++) {\n            \n            for (i = 0; i < newDoc.public[j].length; i++) {\n                if (!newDoc.public[key][i].description) {\n                    throw({forbidden: 'in ' + key + 'you dont have description, and  is required'});\n                }\n                if (!newDoc.public[key][i].url) {\n                    throw({forbidden: 'in ' + key + 'you dont have url, and  is required'});\n                }\nif (!newDoc.public[key][i].type) {\n                    throw({forbidden: 'in ' + key + 'you dont have url, and  is required'});\n                }\n            }\n\n        }\n    }\n\n    if (newDoc.roles && !isArray(newDoc.roles)) {\n        throw({forbidden: 'doc.roles must be an array'});\n    }\n\n}"
</code></pre>

For the last configuration to do, is accept CORS and JSONP request so in the configuraton file of CouchDB change the value to true of 
* allow_jsonp
* enable_cors
