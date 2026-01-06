 // Get IDs of all users present
 var presentUserIds = attendanceQuery
     .Where(a => a.Status == AttendanceStatus.Present)
     .Select(a => a.UserId)
     .Distinct()
     .ToList();

 // Query roles in bulk
 var userRoles = await _userManager.Roles
     .Where(ur => presentUserIds.Contains(ur.UserId))
     .Join(_context.Roles,
           ur => ur.RoleId,
           r => r.Id,
           (ur, r) => new { ur.UserId, r.Name })
     .ToListAsync();

 // Count by role
 int staffPresent = userRoles.Count(ur => ur.Name == Roles.Staff.ToString());
 int studentsPresent = userRoles.Count(ur => ur.Name == Roles.Student.ToString());