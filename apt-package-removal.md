## 🗑️ APT Package Removal Command Comparison:

| Command                          | Removes Package | Removes Config Files | Removes Unused Dependencies |
|----------------------------------|-----------------|-----------------------|-----------------------------|
| `remove <pkg>`                   | ✅              | ✗                     | ✗                           |
| `remove --purge <pkg>`           | ✅              | ✅                    | ✗                           |
| `remove --autoremove <pkg>`      | ✅              | ✗                     | ✅                          |
| `purge --autoremove <pkg>`       | ✅              | ✅                    | ✅                          |
