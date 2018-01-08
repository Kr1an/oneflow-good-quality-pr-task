# Good Quality PR - Oneflow Task
## Initial Code:
```javascript
exports.inviteUser = function(req, res) {
    var invitationBody = req.body;
    var shopId = req.params.shopId;
    var authUrl = "https://url.to.auth.system.com/invitation";
    superagent
        .post(authUrl)
        .send(invitationBody)
        .end(function(err, invitationResponse) {
            if (invitationResponse.status === 201) {
                User.findOneAndUpdate({
                    authId: invitationResponse.body.authId
                }, {
                    authId: invitationResponse.body.authId,
                    email: invitationBody.email
                }, {
                    upsert: true,
                    new: true
                }, function(err, createdUser) {
                    Shop.findById(shopId).exec(function(err, shop) {
                    if (err || !shop) {
                        return res.status(500).send(err || { message: 'No shop found' });
                    }
                    if (shop.invitations.indexOf(invitationResponse.body.invitationId)) {
                        shop.invitations.push(invitationResponse.body.invitationId);
                    }
                    if (shop.users.indexOf(createdUser._id) === -1) {
                        shop.users.push(createdUser);
                    }
                    shop.save();
                    });
                });
            } else if (invitationResponse.status === 200) {
                res.status(400).json({
                    error: true,
                    message: 'User already invited to this shop'
                });
                return;
            }
            res.json(invitationResponse);
        });
};

```
## Analysis:
### Why I do not enjoy this code:
* __possible errors from auth service are not handled__
   * owner is not exist
   * owner does not have access to modify shop
* a lot of __if-else__ statements
* __vars__.
* one __big__ function
### How to make it more clean, testable, exception free, reusable
* __avoid nested structure__ by switching from callback-based to promises and async/await approach. It will allow to have only one try-catch block
* __move each validation step to separate function__. In case of failure each of function throws exception to __common try-catch block__ with specific message
* __use modert js standart__.


