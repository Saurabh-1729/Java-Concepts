This property controls how Hibernate handles database schema generation:

update: Updates the schema based on the entity classes, creating tables if they don't exist
create: Creates the schema, destroying previous data
create-drop: Creates the schema and drops it when the application shuts down
validate: Validates the schema but doesn't make changes
none: No schema actions
