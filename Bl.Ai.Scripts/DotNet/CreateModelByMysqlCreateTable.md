Generate EF Core model creation code, create as follow the example:

## Example 1

Here the mysql create table:
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

Should generate the follow model creating:
```c#
public class UserModel
{
    public int Id { get; set; }
    public string Login { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public bool EmailConfirmed { get; set; }
    public string? PhoneNumber { get; set; }
    public bool PhoneNumberConfirmed { get; set; }
    public bool TwoFactoryEnabled { get; set; }
    public DateTime? LockOutEnd { get; set; }
    public bool LockOutEnabled { get; set; }
    public int AccessFailedCount { get; set; }
    public string Name { get; set; } = string.Empty;
    public string? NickName { get; set; }
    public string Password { get; set; } = string.Empty;
    public string PasswordHash { get; set; } = string.Empty;
    public string PasswordSalt { get; set; } = string.Empty;
    public Guid ConcurrencyStamp { get; set; }
    public bool Active { get; set; } = true;
    public string? ImgUrl { get; set; }
}
```

## Observations
- mysql `char(36)` should be converted to C# `Guid`

## What should be created
So, create a model based on this mysql code:

```sql
-- _add_the_code_here
```