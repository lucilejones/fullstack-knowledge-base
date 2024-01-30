# Associations in-depth

https://guides.rubyonrails.org/association_basics.html

has_one 
-used in the model that contains the primary key. (The foreign key goes on the table for the class declaring the belongs_to association.) It specifies that each instance of the model has one instance of another model. One other model has a reference to this model.
example: a User model has one Profile; in the User model - has_one :profile

belongs_to
-used in the model that is owned by another model. (This model will have the foreign key.)
example: in the Profile model - belongs_to :user

has_many :through
(makes the association indirectly)
-when a record in a table can belong to many records in another table through a third table.
example: a user has many doctors through appointments; a doctor has many users through appointments. A many-to-many associate would be a patient has many doctors and a doctor has many patients.
-this setup makes sense if we need to work with the relationship model as an independent entity (like a rental record that has ids for cars and for renters)

has_and_belongs_to_many
(makes the association directly)
-when a record in a table can belong to many records in another table; this association does not require a third model.
example: an application includes many assemblies and parts, with each assembly having many parts and each part appearing in many assemblies.
-this setup makes sense if we don't need to work with the relationshipw model as an independent entity (though we do need to create the joining table in the database)

polymorphic
-when a record in a table can belong to more than one record from other tables; a model can belong to more than one other model, on a sigle association
example: a comment can belong to a post or a comment can belong to a photo
example: a picture model belongs to either an employee model or a product model

self-join
-when a record in a table can belong to another record in the same table
example: a user can have many friends and a friend can be a user
example: followers and following
example: we want to store all employees in a single database model, but also want to be able to trace relationships such as between manager and subordinates




# Controllers and Routes