+++
archetype = "chapter"
title = "Introduction"
weight = 1
+++

The hiearchy of SQL

- **Databases**: The entire structure of the data system. 
    - **Schemas**: Organization of data elements. 
        - **Tables**: 
            - **Fields**: Holds the data
                - **Data**: 

### Sublanguages

- **Data query language**: Making views of the data.
- **Data definition language**: Modifying structure without affecting contents like cretate and drop.
- **Data control language**: Modifiying permissions.
- **Data manipulation language**: Modifying the data itself like insert, delete, and update.

## Types of databases

### Row-store

Also known as transactional databases because they are effeceint at modifying the data using a row column based system. These databases will serialize an entire row into memory which can be effecient in certain contexts. Due to a row being a single block of memory, width is a primary concern here so row-store is modeled in the third normal form. This kind of model leads to many interacting tables, sometimes with just a few columns. Many tables means more complicated relationships and more joins. For analysis we consolidate the data through denormalization.

###### Primary keys

Unique id that prevents multiple entries. Can also be formed from two columns in what is called a composite key or business key. Tables use primary keys as indicies to make joins easier because only the index needs to be scanned.

### Column-store

Instead of rows as memory blocks, columns are used. Very space effecient by compressing data because columns often have duplicate values we can abbreviate say 100 entries as a compression representation. These kinds of systems do not use primary keys and do not have indicies which can lead to duplicate data if there is not proper care. This structure also helps with analysis because all the data is already in one place.

