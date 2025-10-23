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
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<UserModel>(b =>
    {
        b.ToTable("User");
        b.HasKey(e => e.Id);

        b.Property(e => e.Id)
            .HasColumnName("Id")
            .HasColumnType("int")
            .ValueGeneratedOnAdd()
            .IsRequired();

        b.Property(e => e.Login)
            .HasColumnName("Login")
            .HasColumnType("varchar(250)")
            .IsRequired();

        b.HasIndex(e => e.Login)
            .IsUnique()
            .HasDatabaseName("Login_UNIQUE");

        b.Property(e => e.Email)
            .HasColumnName("Email")
            .HasColumnType("varchar(250)")
            .IsRequired();

        b.Property(e => e.Name)
            .HasColumnName("Name")
            .HasColumnType("varchar(250)")
            .IsRequired();

        b.Property(e => e.NickName)
            .HasColumnName("NickName")
            .HasColumnType("varchar(100)")
            .IsRequired(false);

        b.Property(e => e.Phone)
            .HasColumnName("Phone")
            .HasColumnType("varchar(20)")
            .IsRequired(false);

        b.Property(e => e.EmailConfirmed)
            .HasColumnName("EmailConfirmed")
            .HasColumnType("tinyint(4)")
            .IsRequired();

        b.Property(e => e.PasswordHash)
            .HasColumnName("PasswordHash")
            .HasColumnType("varchar(255)")
            .IsRequired();

        b.Property(e => e.PasswordSalt)
            .HasColumnName("PasswordSalt")
            .HasColumnType("varchar(255)")
            .IsRequired(false);

        b.Property(e => e.LockoutEnd)
            .HasColumnName("LockoutEnd")
            .HasColumnType("datetime")
            .IsRequired(false);

        b.Property(e => e.LockoutEnabled)
            .HasColumnName("LockoutEnabled")
            .HasColumnType("tinyint(4)")
            .HasDefaultValue(true)
            .IsRequired();

        b.Property(e => e.AccessFailedCount)
            .HasColumnName("AccessFailedCount")
            .HasColumnType("int")
            .IsRequired();

        b.Property(e => e.Active)
            .HasColumnName("Active")
            .HasColumnType("tinyint(4)")
            .IsRequired();

        b.Property(e => e.ConcurrencyStamp)
            .HasColumnName("ConcurrencyStamp")
            .HasColumnType("varchar(36)")
            .IsRequired();

        b.Property(e => e.CreatedAt)
            .HasColumnName("CreatedAt")
            .HasColumnType("datetime")
            .HasDefaultValueSql("CURRENT_TIMESTAMP")
            .IsRequired();
    });
}
```

## Observations
- If mysql script allows null column, use `.IsRequired(false)`
- Use the `HasColumnType` for each property
- If model is provided, match the column name with the property model name
- If the model was provided, any property type is `enum` and the mysql column is `varchar`, use the property config `HasConversion<string>()`

## What should be created
So, do the same for this code:

Obs. The model can be included in the code as well


```c#
// _add_the_code_here
```