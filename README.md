graviton [![](https://api.travis-ci.org/emmerge/graviton.svg)](https://travis-ci.org/emmerge/graviton) [ ![Codeship Status for emmerge/graviton](https://codeship.com/projects/54dafd30-7cbe-0132-798d-7a5cdf781b38/status?branch=master)](https://codeship.com/projects/56481)
========

Relations and models for Meteor collections.

Allows you to:

* Define relationships between collections.
* Traverse and retrieve related models.
* Transform your mongo docs into models with attributes and methods.
* Use other mrt packages to handle validation (meteor-simple-schema) hooks (meteor-collection-hooks) and relational pub/sub (reactive-relations).
 
Collections defined with Graviton automatically convert retrieved objects into models. You specify the type(s) when define the collection. Passing `{transform: null}` to `find()` etc. will bypass model transformation. The raw document is stored in model.attributes. Use built-in transformation methods like set, push, pop to make changes to your model locally. Call `model.save()` to persist changes to the database. All methods work on both server and client.

##  Installation

In a Meteor app directory run:

``` sh
$ meteor add emmerge:graviton
```

# API Docs

## Mongo.Collection.prototype
The following are added to all your meteor collections:
* `all()` is an alias for `find().fetch()`
* `build()` returns a new local `Gravition.Model` based on your collection definition. Does not save to db. The instance can be saved using `Model.save()`.
* `create()` calls build() to generate a Model instance then inserts it into the db.

## Graviton
* `Graviton.define(collectionName, options)` Use to define your collections. Returns a Mongo.Collection instantiated with a transform function based on the options passed.

  *Options*
    * `persist` Passing false will define a local collection. defaults to `true`
    * `modelCls` A model constructor generated by calling Graviton.Model.extend
      * or an object like `{key1: ModelClsA, key2: ModelClsB}` if you want polymorphism. `collection.build({_type: 'key2'})` would return an instance of `ModelClsB`.
    * `defaultType` Use when supplying a object for modelCls. Specify the type to use when object being transformed does not contain _type.
    * `timestamps` If `true` and you have the `collection-hooks` package installed, `createdAt` and `updatedAt` timestamps will be added to your models.
    * `<relationName>` Use to define relationships with other collections. See Relations section below.

* `Graviton.getProperty(key)` Use to access deeply-nested attributes without worry of throwing errors. Use period-delimited strings as keys. For example: `var obj = {}; Gravition.getPropery({}, 'some.deep.nested.prop')` would simply return undefined instead of throwing an error because obj.some is undefined. This method works best with keys that are meant to be stored in mongo since periods are not allowed in them.
* `Graviton.setProperty(thing, [value])` Set deeply-nested attributes. Example: `var obj = {}; Gravition.setProperty(obj, 'some.deeply.nested.prop', 'hello')` would result in `{some: {deeply: {nested: {prop: 'hello'}}}}` You can pass an object instead of key, value to set several at once.
* `Graviton.isModel(obj)` Use to check if an object is a model.

## Relations

* belongsTo:
* belongsToMany:
* hasOne:
* hasMany:
* embeds:
* embedsMany:

Pass the following as keys to Graviton.define do declare relationships with other collections. Example: 
```javascript
CarModel = Graviton.Model.extend({
  belongsTo: {
    owner: {
      klass: 'people',
      foreignKey: 'ownerId'
    }
  },
  belongsToMany: {
    drivers: {
      klass: 'drivers',
      field: 'driverIds'
    }
  },
  hasOne: {
    // note that manufacturer should really be a belongsTo relationship since manufacturer shouldn't have a single carId
    manufacturer: {
      klass: 'manufacturers',
      foreignKey: 'carId'
    }
  },
  hasMany: {
    wheels: {
      klass: 'wheels',
      foreignKey: 'carId'
    }
  },
  embeds: {
    plate: {
      klass: 'plates'
    }
  },
  embedsMany: {
    windows: {
      klass: 'windows'
    }
  }
},{});

Car = Graviton.define("cars", {
  modelCls: CarModel
});
```
Would make the following possible:
```javascript
var car = Car.findOne();
car.owner(); // returns a model
car.owner(person); // sets the ownerId of person

car.wheels.find(); // returns a cursor. same as Wheel.find({carId: car._id})
car.wheels.add({}); // insert a document into the wheels collection with carId = car._id

car.drivers.find(); // returns a cursor. same as Driver.find({_id: {$in: car.get('driverIds')}})

car.manufacturer();
car.plate();

// embedded models don't have the same finder capability since they aren't kept in minimongo
car.windows.all(); // returns all models
car.windows.at(2); // only builds one model
```

## Graviton.Model

Graviton transforms collection objects into Models for you. This allows them to carry useful metadata and functions with the data. The vanilla Graviton.Model allows for basic functionality. Defining an extension of the Graviton.Model allows you to specify details of the collection's relationships or other custom functionality.

* `Graviton.Model.extend({Options}, {ExtensionPrototype});` Use to define your collections. Returns a Mongo.Collection instantiated with a transform function based on the options passed.

  *Options*
    * `defaults`: an object containing default key:value pairs for the collection. These key:values will be added to all model instances where there is not already a stored value with the same key. Functions should not be placed here as stored records cannot have functions as values.
    * `initialize`: a function which will be run on initialization.
    * _relationships_: define how this model relates to other collections. `belongsTo`, `belongsToMany`, `hasOne`, `hasMany`, etc.
    
  *ExtensionPrototype*
    * This object contains the extension prototype. This is the place to add functions to the model. Values could also be placed here if they relate to this specific model. These do not behave as attributes - any values placed here will not be stored.
      

# Examples

### full app example

[Graviscope](https://github.com/mhwheeler/Graviscope) - a graviton version of the Microscope app built in the book Discover Meteor

### Working with relations

### `Model.hasMany.add()` example (simplified from graviscope)

```javascript
PostModel = Graviton.Model.extend({
  hasMany: {
    comments: {
      collection: 'comments',
      foreignKey: 'postId'
    }
  }
},{};

Posts = Graviton.define('posts', {
  modelCls: PostModel
});
```

```javascript
post = Posts.findOne();
comment = Comments.build();
comment.set({author:'Steve', body:'Check out the Posts.hasMany.comments relationship.'})
post.comments.add(comment);
console.log('Newly created comment id', comment._id);
```

### `belongsTo` examples (simplified from graviscope)
```javascript

```


