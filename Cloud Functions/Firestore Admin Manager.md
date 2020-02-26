## TASK: Add a custom claim to a user based on a firestore document
No more need to create one time websites, console commands, or exploitable interfaces to add roles to users
Manage it directly inside a secured firestore document.

## How to use:
Insert this into your cloud functions server, hosted on your local firebase cloud, or in your dedicated server.
To Configure, set the path in .document("path") to the relavent location where this information would be held

## WARNING: 
 It is HIGHLY important that access to this document, and other relative information is not accessible to the public
 this can be secured with rules (see below: Rules) and the appropriate health checks.
 `context.auth` is an unreliable source for auth confirmations from the console and adminSdk as they return 'undefined'
 It should also be noted about caveats of finding users by email, it would be prefered to find a user by UID instead.
 refer to the documentation: https://firebase.google.com/docs/auth/admin/manage-users#retrieve_user_data


## SNIPPET: Cloud Function (manageAdmins)
```
exports.manageAdmins = functions.firestore
    .document('Config/Admins')
    .onWrite((change, context) => {
        // compare before and after, concat the change
        const before = change.before.data()!;
        const after = change.after.data()!;
        Object.keys(before).forEach(key => {
            if (before[key] === true && (after[key]) !== true)
                admin.auth().getUserByEmail(key)
                    .then(function (userRecord) {
                        ChangeAdmin(userRecord.uid, false)
                            .catch(function (error) {
                                console.error('Error Changing Admin Perms:', error);
                            });
                        console.log('Successfully fetched user data:', userRecord.toJSON());
                    })
                    .catch(function (error) {
                        console.log('Error fetching user data:', error);
                    });
        });
        // Promote New Admins
        Object.keys(after).forEach(key => {
            if ((before[key]) !== true && after[key] === true)
                admin.auth().getUserByEmail(key)
                    .then(function (userRecord) {
                        ChangeAdmin(userRecord.uid, true)
                            .catch(function (error) {
                                console.error('Error Chanting Admin Perms:', error);
                            });
                        console.log('Successfully fetched user data:', userRecord.toJSON());
                    })
                    .catch(function (error) {
                        console.log('Error fetching user data:', error);
                    });
        });
        return true;
    });
```

### SNIPPET: Rules
Example Rules to secure the below app from all external access
make sure the `/Config/Admins` matches your above firestore structure
denying all access, ensures only the adminSdk and the console can edit this data.
```
service cloud.firestore {
  match /databases/{database}/documents {
    match /Config/Admins {
      allow read, write: if false;
    }
    // all other rules below, so no conflict can occur
  }
}
```
