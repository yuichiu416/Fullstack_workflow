### Store Objects in the model
1. In the `model.js`, put `Map` as the type.
   Ex. 
   ```json
       nameHash: {
        type: Map,
    }
    ```
2. In the `type.js`, put `GraphQLString` as the type, and resolve it accordingly
   Ex.
   ```js
    nameHash: { 
        type: GraphQLString,
        resolve(parentValue){
            if (parentValue.nameHash) {
                return JSON.stringify(parentValue.nameHash.toJSON());
            }
        } 
    }
    ```
    `JSON.stringify()`/`JSON.parse()` it or `.toJSON()` whenever necessary.