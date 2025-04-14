``` javascript
function getDataFromIndex() {
  return new Promise((resolve, reject) => {
    const dbName = "ngStorage";
    const request = indexedDB.open(dbName);

    request.onerror = (event) => {
      console.error("IndexedDB error:", event.target.error);
      reject(event.target.error);
    };

    request.onsuccess = (event) => {
      const db = event.target.result;
      const allData = {};


      const objectStoreNames = Array.from(db.objectStoreNames);

      if (objectStoreNames.length === 0) {
        console.log("No object stores found in the database");
        resolve(allData);
        return;
      }

      let completedStores = 0;

      objectStoreNames.forEach((storeName) => {
        allData[storeName] = {};

        const transaction = db.transaction(storeName, "readonly");
        const objectStore = transaction.objectStore(storeName);
        const cursorRequest = objectStore.openCursor();

        cursorRequest.onsuccess = (event) => {
          const cursor = event.target.result;
          if (cursor) {
            allData[storeName][cursor.key] = cursor.value;
            cursor.continue();
          }
        };

        transaction.oncomplete = () => {
          console.log(`Data from store '${storeName}':`, allData[storeName]);
          completedStores++;

          if (completedStores === objectStoreNames.length) {
            console.log("All data from IndexedDB:", allData);
            resolve(allData);
          }
        };

        transaction.onerror = (event) => {
          console.error(`Error in store '${storeName}':`, event.target.error);
          reject(event.target.error);
        };
      });
    };

    request.onupgradeneeded = (event) => {
      console.log("Database upgrade needed");
      const db = event.target.result;
      resolve({});
    };
  });
}
```

``` javascript
getDataFromIndex()
  .then((data) => {
    console.log("Successfully retrieved all data");
    
  })
  .catch((error) => {
    console.error("Failed to retrieve data:", error);
  });
```