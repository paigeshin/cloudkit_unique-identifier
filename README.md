# cloudkit_unique-identifier

### Capability

- CloudKit 

### Add User Indexes to CloudKitContainer

- recordName, Queryable

```swift
class CloudKitHandler: ObservableObject {
    
    @Published var isSignedInToiCloud: Bool = false
    @Published var recordId: String = ""
    @Published var userName: String = ""
 
    func requestPermission() {
        CKContainer.default().requestApplicationPermission([.userDiscoverability]) { status, error in
            if let error {
                self.isSignedInToiCloud = false
                print(error.localizedDescription)
                return
            }
            switch status {
            case .initialState:
                break
            case .couldNotComplete:
                break
            case .denied:
                DispatchQueue.main.async {
                    self.isSignedInToiCloud = false
                }
            case .granted:
                DispatchQueue.main.async {
                    self.isSignedInToiCloud = true
                }
            @unknown default:
                break
            }
        }
    }
    
    func getiCloudStatus() {
        CKContainer.default().accountStatus { status, error in
            if let error {
                self.isSignedInToiCloud = false
                print(error.localizedDescription)
                return
            }
            
            DispatchQueue.main.async {
                switch status {
                case .available:
                    self.isSignedInToiCloud = true
                case .noAccount:
                    self.isSignedInToiCloud = false
                case .couldNotDetermine:
                    self.isSignedInToiCloud = false
                case .restricted:
                    self.isSignedInToiCloud = false
                default:
                    self.isSignedInToiCloud = false
                }
            }
        }
    }

    func fetchiCloudUserRecordID() {
        CKContainer.default().fetchUserRecordID { id, error in
            if let error {
                print(error.localizedDescription)
                return
            }
            if let id {
                DispatchQueue.main.async {
                    self.recordId = id.recordName
                }
                self.discoveriCloudUser(id: id)
            }
        }
    }
    
    func discoveriCloudUser(id: CKRecord.ID) {
        CKContainer.default().discoverUserIdentity(withUserRecordID: id) { identity, error in
            DispatchQueue.main.async {
                if let name = identity?.nameComponents?.givenName {
                    self.userName = name
                }
            }
        }
    }
    
    
}
```
