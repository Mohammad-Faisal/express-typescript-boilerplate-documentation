---
sidebar_position: 7
---

# Project Structure

If you want to add a new route then you will goto `/routes` folder and add a new Router.
Then register that router in the `index.ts` file under the `/routes` folder.

Then you will create a Controller under the `/controllers` directory.All business logics should go into there.

Specific use cases should be handles by Service classes under the `/service` folder.

All Database related things should go under `/repositories` folder.

To create a new model for data base look into the `/models` folder.
