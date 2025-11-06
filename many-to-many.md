## 🧩 The Concept

You want:

* A **User** that can have multiple **Skills**
* A **Skill** that can belong to multiple **Users**

That means we’ll have:

```
User ⇄ UserSkill ⇄ Skill
```

The middle table, `UserSkill`, is your **join entity**.

---

## 🏗️ Entity Models (EF Core Style)

### User.cs

```csharp
public class User
{
    public Guid Id { get; set; }
    public string FullName { get; set; } = string.Empty;

    // Navigation property
    public ICollection<UserSkill> UserSkills { get; set; } = new List<UserSkill>();
}
```

### Skill.cs

```csharp
public class Skill
{
    public Guid Id { get; set; }
    public string Name { get; set; } = string.Empty;

    // Navigation property
    public ICollection<UserSkill> UserSkills { get; set; } = new List<UserSkill>();
}
```

### UserSkill.cs (Join Table)

```csharp
public class UserSkill
{
    public Guid UserId { get; set; }
    public User User { get; set; } = null!;

    public Guid SkillId { get; set; }
    public Skill Skill { get; set; } = null!;

    public DateTime AddedAt { get; set; } = DateTime.UtcNow;
}
```

---

## ⚙️ EF Core Configuration

If you’re using Fluent API:

```csharp
modelBuilder.Entity<UserSkill>()
    .HasKey(us => new { us.UserId, us.SkillId });

modelBuilder.Entity<UserSkill>()
    .HasOne(us => us.User)
    .WithMany(u => u.UserSkills)
    .HasForeignKey(us => us.UserId);

modelBuilder.Entity<UserSkill>()
    .HasOne(us => us.Skill)
    .WithMany(s => s.UserSkills)
    .HasForeignKey(us => us.SkillId);
```

That’s all you need — EF Core will create a `UserSkills` table with a composite key of `(UserId, SkillId)`.

---

## 🧠 Example Usage

### Add a skill to a user

```csharp
var user = await _context.Users.Include(u => u.UserSkills).FirstAsync(u => u.Id == userId);
var skill = await _context.Skills.FirstAsync(s => s.Id == skillId);

user.UserSkills.Add(new UserSkill { User = user, Skill = skill });
await _context.SaveChangesAsync();
```

### Query user’s skills

```csharp
var skills = await _context.UserSkills
    .Where(us => us.UserId == userId)
    .Select(us => us.Skill.Name)
    .ToListAsync();
```

### Query users with a specific skill

```csharp
var users = await _context.UserSkills
    .Where(us => us.Skill.Name == "C#")
    .Select(us => us.User.FullName)
    .ToListAsync();
```

---

## ✨ Optional Shortcut (if you’re on EF Core 5+)

You can *skip* the join entity if you don’t need extra fields like `AddedAt`:

```csharp
public class User
{
    public Guid Id { get; set; }
    public string FullName { get; set; } = string.Empty;
    public ICollection<Skill> Skills { get; set; } = new List<Skill>();
}

public class Skill
{
    public Guid Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public ICollection<User> Users { get; set; } = new List<User>();
}

modelBuilder.Entity<User>()
    .HasMany(u => u.Skills)
    .WithMany(s => s.Users)
    .UsingEntity(j => j.ToTable("UserSkills"));
```