Create a MYSQL create table script based on a model, I'll give you an example:

## C# Model:
```c#
public class UserModel
{
    public int Id { get; set; }
    public string Login { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public string Name { get; set; }
    public string? NickName { get; set; }
    public string? Phone { get; set; }
    public bool EmailConfirmed { get; set; }
    public string PasswordHash { get; set; } = string.Empty;
    public string? PasswordSalt { get; set; }
    public DateTime? LockoutEnd { get; set; }
    public bool LockoutEnabled { get; set; }
    public int AccessFailedCount { get; set; }
    public bool Active { get; set; }
    public bool BlockedAccess { get; set; }
    public string ImgUrl { get; set; }
    public Guid? ConcurrencyStamp { get; set; }
    public DateTime CreatedAt { get; set; }
}
```

Should generate the follow script:
```sql
CREATE TABLE `User` (
   `Id` int NOT NULL AUTO_INCREMENT,
   `Login` varchar(250) NOT NULL,
   `Email` varchar(250) NOT NULL,
   `Name` varchar(250) NOT NULL,
   `NickName` varchar(100) DEFAULT NULL,
   `Phone` varchar(20) DEFAULT NULL,
   `EmailConfirmed` tinyint NOT NULL,
   `PasswordHash` varchar(255) NOT NULL,
   `PasswordSalt` varchar(255) DEFAULT NULL,
   `LockoutEnd` datetime DEFAULT NULL,
   `LockoutEnabled` tinyint NOT NULL DEFAULT '1',
   `AccessFailedCount` int NOT NULL,
   `Active` tinyint NOT NULL,
   `BlockedAccess` tinyint DEFAULT '0',
   `ImgUrl` varchar(500) DEFAULT NULL,
   `ConcurrencyStamp` varchar(36) NOT NULL,
   `CreatedAt` datetime DEFAULT CURRENT_TIMESTAMP,
   PRIMARY KEY (`Id`),
   UNIQUE KEY `Uq_User_Login` (`Login`)
 ) ENGINE=InnoDB
```

## Observations
- If the model is `string`, the mysql type should be `varchar(250)`
- If the model is `guid`, the mysql type should be `char(36)`
- If the property has comments, add it into the column mysql comments
- If the property contains 'Json' in the name or documentantion, set the mysql type as 'text CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci'

## What should be created

Now generate a mysql create table script for the follow model:

```c#
/*_add_your_model_here*/
```