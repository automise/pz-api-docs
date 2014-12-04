PayZip API Documentation
===
PayZip v1.1 API v2.2- 7/1/2015

Introduction
---

The following documentation outlines the PayZip API and how it can be used to interact with organisations, members, invoices and remittance.


Authentication
---

#### HTTP Authentication
The PayZip API can only be accessed through HTTPS and requires Basic HTTP Authentication using the following Username and Password.  

`Username: Supplied separately`  
`Password: Supplied separately`  

The Username and Password should be colon seperated and then Base64 Encoded, the word Basic should then be prepended and included as an Authorization header for the request.

For example:

`Authorization: Basic MTUz2ipRN3R4YUZy`


#### Signature
Each request to the API must be signed with a signature, this signature differs dependent on each request. The signature is a SHA256 hash known as an HMAC. 

This signature depends on a set of values and a user key, your user key can be found below.  

`User Key: Supplied separately`

Here's a sample function in C# for creating a HMAC.

```
public static string CreateHMAC(string dataToHash, string key)
{
    if(!String.IsNullOrWhiteSpace(dataToHash) && !String.IsNullOrWhiteSpace(key))
    {
        var dataBytes = Encoding.UTF8.GetBytes(dataToHash);
        var keyBytes = Encoding.UTF8.GetBytes(key);

        using(var hmac = new HMACSHA256(keyBytes))
        {
            var hash = hmac.ComputeHash(dataBytes);
            return Convert.ToBase64String(hash);
        }
    }

    return string.Empty;
}
```

Creating Organisations
---

* `POST /api/organisation`  

```
{  
    "FirstName": "David",  
    "Surname": "Jones",  
    "Email": "david.jones@mytestclub.co.uk",  
    "OrganisationName": "My Organisation",  
    "AbsorbFees": true,  
    "Signature": "HMAC Signature"  
}  
```

This will return `201 Created` and the ID of the new Organisation a JSON Response.  

### Creating the signature  
The Organisation HMAC signature is created as follows `CreateHMAC(OrganisationName + Email, userKey);`


Creating Members
---

* `POST /api/member`  

```
{  
    "OrganisationId": 233,  
    "Name": "David Jones",  
    "Email": "david.jones@mytestclub.co.uk",  
    "Signature": "HMAC Signature"  
}
```

### Creating the signature  
The Members HMAC signature is created as follows `CreateHMAC(OrganisationId.ToString() + Name + Email, userKey);`

This will return `201 Created` and the ID of the new member as a JSON Response.

Retrieving a MemberID
---
Returns the ID of a member.

* `GET /api/member/?memberName=Name&memberEmail=email@email.com&orgId=2`

```
6140
```

Updating Members
---
* `POST /api/memberupdate`  

Supply the member ID along with any information you would like to change and the signature.  

```
{  
    "MemberId": 6009,  
    "Name": "David Jones Sr",  
    "Signature": "HMAC Signature"  
}  
```

This will return `204 No Content`

### Creating the signature  
The Members HMAC signature is created as follows `CreateHMAC(OrganisationId.ToString() + Name + Email, userKey);`


Adding Bank Details
---
* `GET /api/bankdetails/?id=1`  

```
{  
    BankDetailUrls: https://localhost:10504/Organisation/BankDetails?enc=ecisHKgve2Zga8IqtQk6ay%2fCDEQIxYgIZtbcSpQHUtp4V0XdISjZFwHqCAImp0OG  
}
```  

An OrganisationId is required in the `GET` request, it returns a URL that expires 10 minutes from creation.  

Creating an Invoice
---
* `POST /api/invoice`  

```
{  
    "InvoiceTitle": "Test Invoice",
    "OrganisationAbsorbsFees": true,
    "DueDate": '2014-02-01 00:00:00',
    "OrganisationId": 276,
    "Signature": "HMAC Signature",
    "Invoices":[
    {
        "MemberId":4975,
        "ThirdParty":"True",
        "ThirdPartyFeeAmount":10,
        "ThirdPartyDescription": “UKA Fee",
        "ThirdPartyOrgId":113,
        "Items":[
        {
            "Description":"Annual Subscription",
            "Amount":70
        },
        ]
    },
    {
        "MemberId":4976,
        "ThirdParty":"True",
        "ThirdPartyFeeAmount":15,
        "ThirdPartyOrgId":113,
        "Items":[
        {
            "Description":"Annual Subscription",
            "Amount":45.1
        },
        {
            "Description":"Registration Fee",
            "Amount":10
        }
        ]
    } 
    ]
}
```

Returns `200 OK` and the following  

```
{
    "id":17298,
    "MemberId":4981,
    "PaymentURL":"https://localhost:10504/Email/ViewAndPayRequest?enc=Nwdy6IaxDK8T96OErIWBPg%3d%3d"
},
{
    "id":17299,
    "MemberId":4982,
    "PaymentURL":"https://localhost:10504/Email/ViewAndPayRequest?enc=rDOZN0ELz2BCkBcQgmwrRQ%3d%3d"
}
```
### Creating the signature  
The invoice HMAC signature is created as follows `CreateHMAC(DueDate.ToString('yyy-MM-dd HH:mm:ss') + InvoiceTitle + OrganisationAbsorbsFees + OrganisationId + Invoices, userKey);`
Where `Invoices` is a concatenated string of values, as follows:
```
string Invoices = "";
foreach(var invoices in model.Invoices)
{
    Invoices += invoices.MemberId.ToString();
    Invoices += invoices.ThirdParty.ToString();
    Invoices += invoices.ThirdPartyDescription.ToString();
    Invoices += invoices.ThirdPartyFeeAmount.ToString();
    Invoices += invoices.ThirdPartyOrgId.ToString();
    foreach(var item in invoices.Items)
    {
        Invoices += item.Amount.ToString();
        Invoices += item.Description.ToString();
    }
}
```

Get Invoice 
---
* `GET /api/invoice/?id=`

Supply an InvoiceID to receive the following information

```
{
    InvoiceID: 1234,
    MemberID: 1234,
    OrganisationId: 123,
    Amount: 20.24,
    Paid: true,
    DatePaid: 2014-02-01 00:00:00
}
```
Post Payment Webhook
---
The Postpayment Webook sends the following information the the supplied address:
```
{
    InvoiceId: 1234,
    PaymentStatus: "AUTHORISED"
}
```
### Creating the signature
The HMAC is created as follows: `CreateHMAC(InvoiceId.ToString() + PaymentStatus, userKey);`

Get Remittance IDs
---
* `GET /api/remittance/?orgId=0`  

Send an OrganisationID to receive a list of BACS IDs for that Organisation

Get Remittance Detail
---
* `GET /api/organisationremittance/?BacsId=0&OrgId=0`

Send a BACSID and an OrganisationId to retrieve the remittance detail.

Returns `200 OK` and the following

```
{
    "OrganisationId": 123,
    "TotalAmout": 200.00,
    "Date": "2015-01-01 00:00:00.000",
    "Payments": [
        {
            "InvoiceId": 6005,
            "TotalPaid": 200.00,
            "TotalFees": 10.00,
            "TotalClub": 190.00
        }
    ]
}  
```
