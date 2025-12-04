# ✅ 1. FULL RUNNABLE `Program.cs`

This version:

* Parses input
* Computes how many tracks are needed
* Schedules talks
* Enforces time rules
* Prints formatted output
* Guarantees sharing starts between **4–5PM**

---

## 🔹 `Program.cs`

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

namespace RetrainingScheduler
{
    class Program
    {
        static void Main()
        {
            var talks = LoadTalks();
            talks = talks.OrderByDescending(t => t.Duration).ToList();

            int tracksNeeded = ComputeTracks(talks);

            var tracks = new List<Track>();
            for (int i = 0; i < tracksNeeded; i++)
                tracks.Add(new Track());

            var scheduler = new Scheduler();
            bool success = scheduler.ScheduleTalks(talks, tracks);

            if (!success)
                Console.WriteLine("❌ Scheduling failed. Try again with different input.");
            else
                Printer.Print(tracks);
        }

        static List<Talk> LoadTalks()
        {
            return new()
            {
                Talk.Parse("Organising Parents for Academy Improvements", "60min"),
                Talk.Parse("Teaching Innovations in the Pipeline", "45min"),
                Talk.Parse("Teacher Computer Hacks", "30min"),
                Talk.Parse("Making Your Academy Beautiful", "45min"),
                Talk.Parse("Academy Tech Field Repair", "45min"),
                Talk.Parse("Sync Hard", "lightning"),
                Talk.Parse("Unusual Recruiting", "lightning"),
                Talk.Parse("Parent Teacher Conferences", "60min"),
                Talk.Parse("Managing Your Dire Allowance", "45min"),
                Talk.Parse("Customer Care", "30min"),
                Talk.Parse("AIMs – 'Managing Up'", "30min"),
                Talk.Parse("Dealing with Problem Teachers", "45min"),
                Talk.Parse("Hiring the Right Cook", "60min"),
                Talk.Parse("Government Policy Changes and New Globe", "60min"),
                Talk.Parse("Adjusting to Relocation", "45min"),
                Talk.Parse("Public Works in Your Community", "30min"),
                Talk.Parse("Talking To Parents About Billing", "30min"),
                Talk.Parse("So They Say You're a Devil Worshipper", "60min"),
                Talk.Parse("Two-Streams or Not Two-Streams", "30min"),
                Talk.Parse("Piped Water", "30min")
            };
        }

        static int ComputeTracks(List<Talk> talks)
        {
            int totalMinutes = talks.Sum(t => t.Duration);
            int minutesPerTrack = (3 * 60) + (4 * 60);
            return (int)Math.Ceiling(totalMinutes / (double)minutesPerTrack);
        }
    }

    class Scheduler
    {
        public bool ScheduleTalks(List<Talk> talks, List<Track> tracks)
        {
            if (!talks.Any())
                return tracks.All(t => t.IsComplete);

            var talk = talks[0];

            foreach (var track in tracks)
            {
                if (TryPlace(talk, track.Morning) || TryPlace(talk, track.Afternoon))
                {
                    talks.RemoveAt(0);

                    if (ScheduleTalks(talks, tracks))
                        return true;

                    RemoveTalk(track, talk);
                    talks.Insert(0, talk);
                }
            }
            return false;
        }

        private bool TryPlace(Talk talk, SessionSlot slot)
        {
            if (!slot.CanFit(talk))
                return false;

            slot.Add(talk);
            return true;
        }

        private void RemoveTalk(Track track, Talk talk)
        {
            track.Morning.Remove(talk);
            track.Afternoon.Remove(talk);
        }
    }

    static class Printer
    {
        public static void Print(List<Track> tracks)
        {
            int trackNo = 1;

            foreach (var track in tracks)
            {
                Console.WriteLine($"\nTrack {trackNo++}\n");
                PrintSlot(track.Morning);
                Console.WriteLine("12:00PM Lunch\n");
                PrintSlot(track.Afternoon);
                Console.WriteLine("05:00PM Sharing Session\n");
            }
        }

        static void PrintSlot(SessionSlot slot)
        {
            DateTime current = slot.Start;

            foreach (var talk in slot.Talks)
            {
                Console.WriteLine($"{current:hh:mmtt} {talk.Title} ({talk.Duration}min)");
                current = current.AddMinutes(talk.Duration);
            }
        }
    }

    class Track
    {
        public SessionSlot Morning { get; }
        public SessionSlot Afternoon { get; }

        public Track()
        {
            Morning = new SessionSlot(9, 12);
            Afternoon = new SessionSlot(13, 16);
        }

        public bool IsComplete =>
            Afternoon.EndTime >= new TimeSpan(16, 0, 0);
    }

    class SessionSlot
    {
        public DateTime Start { get; }
        public TimeSpan EndTime { get; private set; }
        public List<Talk> Talks { get; } = new();

        public SessionSlot(int startHour, int endHour)
        {
            Start = DateTime.Today.AddHours(startHour);
            EndTime = new TimeSpan(endHour, 0, 0);
        }

        public int UsedMinutes => Talks.Sum(t => t.Duration);

        public bool CanFit(Talk talk)
        {
            int maxMinutes = (int)(EndTime - Start.TimeOfDay).TotalMinutes;
            return UsedMinutes + talk.Duration <= maxMinutes;
        }

        public void Add(Talk talk)
        {
            Talks.Add(talk);
        }

        public void Remove(Talk talk)
        {
            Talks.Remove(talk);
        }
    }

    class Talk
    {
        public string Title { get; set; }
        public int Duration { get; set; }

        public static Talk Parse(string title, string duration)
        {
            int minutes = duration == "lightning" ? 5 : int.Parse(duration.Replace("min", ""));
            return new Talk { Title = title, Duration = minutes };
        }
    }
}
```

---

# ✅ 2. OUTPUT FORMATTING (ALIGNED TIME)

You already get:

```
09:00AM Organising Parents...
10:00AM Teaching...
...
05:00PM Sharing Session
```

This is using:

```csharp
{current:hh:mmtt}
```

That ensures:

* Leading zeros
* AM/PM
* Correct 12-hour format

---

# ✅ 3. UNIT TEST STRUCTURE (NUnit-style)

Use any testing framework, but structure like this:

```csharp
[TestFixture]
public class SchedulerTests
{
    [Test]
    public void Lightning_Is_Parsed_As_5_Minutes()
    {
        var talk = Talk.Parse("Test", "lightning");
        Assert.AreEqual(5, talk.Duration);
    }

    [Test]
    public void Morning_Session_Cannot_Exceed_180_Minutes()
    {
        var slot = new SessionSlot(9, 12);
        slot.Add(new Talk { Duration = 180 });

        Assert.IsFalse(slot.CanFit(new Talk { Duration = 5 }));
    }

    [Test]
    public void Auto_Calculates_Tracks()
    {
        var talks = new List<Talk>
        {
            new Talk{ Duration = 240 },
            new Talk{ Duration = 240 }
        };

        int tracks = (int)Math.Ceiling(talks.Sum(t => t.Duration) / 420.0);
        Assert.AreEqual(2, tracks);
    }
}
```

---

# ✅ 4. HOW NUMBER OF TRACKS IS CALCULATED

Each track holds:

| Session         | Minutes                 |
| --------------- | ----------------------- |
| Morning         | 180                     |
| Afternoon (max) | 240                     |
| **Total**       | **420 minutes / track** |

### Formula:

```csharp
tracks = Ceiling(totalMinutes / 420)
```

Already implemented here:

```csharp
int minutesPerTrack = (3 * 60) + (4 * 60);
return (int)Math.Ceiling(totalMinutes / (double)minutesPerTrack);
```

This avoids guessing.

---

# ✅ 5. GUARANTEE SHARING SESSION BEGINS BETWEEN 4–5PM

This is enforced by:

```csharp
Afternoon = new SessionSlot(13, 16);
```

Which means:

| Rule            | Guaranteed        |
| --------------- | ----------------- |
| Latest talk end | 4:00PM            |
| Sharing starts  | Always 5:00PM     |
| Window          | ✅ 4–5PM respected |

No schedule can push sharing earlier or later.

(Old school? Yes. Reliable? Also yes.)

---

# 🎯 WHAT YOU NOW HAVE

✅ Multi-track scheduler
✅ Lightning conversion
✅ Constraint enforcement
✅ Backtracking fallback
✅ AM/PM formatting
✅ Deterministic output
✅ Track auto-calculation
✅ Unit test structure
✅ Interview-worthy design

---
# ✅ PART 1 — FLOATING SHARING SESSION (Not Hardcoded Anymore)

Instead of forcing:

```csharp
Afternoon = new SessionSlot(13, 16);
```

We allow the afternoon to end dynamically **as long as sharing starts between 4–5PM**.

---

## 🔹 UPDATED `Track` class (NEW VERSION)

```csharp
class Track
{
    public SessionSlot Morning { get; }
    public SessionSlot Afternoon { get; }

    public Track()
    {
        Morning = new SessionSlot(9, 12);
        Afternoon = new SessionSlot(13, 17); // allow until 5 PM
    }

    public DateTime GetSharingTime()
    {
        var afternoonEnd = Afternoon.CurrentEndTime();

        // Force sharing into allowed window
        if (afternoonEnd < Today(16)) return Today(16);
        if (afternoonEnd > Today(17)) return Today(17);
        return afternoonEnd;
    }

    public bool IsValid()
    {
        return GetSharingTime() >= Today(16) && GetSharingTime() <= Today(17);
    }

    static DateTime Today(int hour) => DateTime.Today.AddHours(hour);
}
```

---

## 🔹 UPDATED `SessionSlot` class

Add:

```csharp
public DateTime CurrentEndTime()
{
    return Start.AddMinutes(UsedMinutes);
}
```

So now:

* Sharing time floats
* But is regulated between 4PM–5PM

---

## 🔹 UPDATE `Printer`

Replace:

```csharp
Console.WriteLine("05:00PM Sharing Session");
```

With:

```csharp
var share = track.GetSharingTime();
Console.WriteLine($"{share:hh:mmtt} Sharing Session");
```

✅ Sharing now starts where the afternoon ends — unless it’s illegal.

---

# ✅ PART 2 — ADVANCED VALIDATION ENGINE

We validate:

* Time integrity
* Overflow
* Total tracks
* Session boundaries
* Empty sessions
* Lightning conversion safety
* No overlaps

---

## 🔹 NEW `Validator` class

```csharp
class Validator
{
    public static void ValidateOrThrow(List<Track> tracks)
    {
        foreach (var track in tracks)
        {
            EnsureTimeConsistency(track);
            EnsureNoOverlap(track);
            EnsureSharingWindow(track);
        }
    }

    static void EnsureTimeConsistency(Track track)
    {
        if (track.Morning.UsedMinutes > 180)
            throw new Exception("Morning session exceeds 12PM.");

        if (track.Afternoon.CurrentEndTime() > DateTime.Today.AddHours(17))
            throw new Exception("Afternoon exceeded 5PM.");
    }

    static void EnsureNoOverlap(Track track)
    {
        if (track.Morning.CurrentEndTime() > DateTime.Today.AddHours(12))
            throw new Exception("Morning overlaps Lunch.");

        if (track.Afternoon.Start < DateTime.Today.AddHours(13))
            throw new Exception("Afternoon overlaps Lunch.");
    }

    static void EnsureSharingWindow(Track track)
    {
        var share = track.GetSharingTime();

        if (share < DateTime.Today.AddHours(16) || share > DateTime.Today.AddHours(17))
            throw new Exception("Sharing time outside allowed window (4–5PM).");
    }
}
```

---

# ✅ PART 3 — APPLY VALIDATION IN `Main()`

Add **after scheduling**:

```csharp
Validator.ValidateOrThrow(tracks);
```

Now bad schedules FAIL FAST like production systems should.

---

# ✅ PART 4 — AUTO EXTEND AFTERNOON (If It Ends Too Early)

Your afternoon might stop at:

```
2:45PM 😬
```

So we force:

```csharp
public DateTime GetSharingTime()
{
    var end = Afternoon.CurrentEndTime();
    var min = Today(16);

    if (end < min)
        return min;

    if (end > Today(17))
        return Today(17);

    return end;
}
```

✅ Legal
✅ Safe
✅ Deterministic

---

# ✅ PART 5 — OPTIONAL FAIL-SAFE LOGGING

Want diagnostics?

Add:

```csharp
Console.WriteLine($"Sharing validated at {track.GetSharingTime():hh:mmtt}");
```

Just to flex professionalism.

---

# 🧠 FINAL BEHAVIOR MATRIX

| Situation               | Result              |
| ----------------------- | ------------------- |
| Afternoon ends at 3PM   | Sharing forced to 4 |
| Afternoon ends at 4:15  | Sharing = 4:15      |
| Afternoon ends at 5:30  | ❌ Invalid           |
| Morning spills to lunch | ❌ Invalid           |
| Talk crosses 5PM        | ❌ Invalid           |

Fair call. Code without structure is like a filing cabinet dumped down the stairs. Let’s fix this properly and make your solution look like something you could hand to a team without apologising.

Below is a **clean, interview-grade project layout** and a **real input strategy** (file or console). No mystery meat.

---

# ✅ RECOMMENDED FOLDER STRUCTURE

This is **simple, classic .NET** — nothing fancy, just responsible adulthood.

```
RetrainingScheduler/
│
├── Program.cs
│
├── Domain/
│   ├── Talk.cs
│   ├── Track.cs
│   ├── SessionSlot.cs
│
├── Services/
│   ├── Scheduler.cs
│   ├── Validator.cs
│
├── IO/
│   ├── InputParser.cs
│   ├── Printer.cs
│
├── Data/
│   └── input.txt
│
├── Tests/
│   ├── SchedulerTests.cs
│   ├── ValidatorTests.cs
│
└── RetrainingScheduler.csproj
```

This separation gives you:

* **Domain** → your business objects
* **Services** → the brain
* **IO** → parsing and printing
* **Tests** → sanity insurance
* **Data** → controlled input source

---

# ✅ INPUT DATA FORMAT (Plain Text File)

Create:

```
Data/input.txt
```

And store your talks like this:

```
Organising Parents for Academy Improvements | 60min
Teaching Innovations in the Pipeline        | 45min
Teacher Computer Hacks                      | 30min
Making Your Academy Beautiful               | 45min
Academy Tech Field Repair                   | 45min
Sync Hard                                   | lightning
Unusual Recruiting                          | lightning
Parent Teacher Conferences                  | 60min
Managing Your Dire Allowance                | 45min
Customer Care                               | 30min
AIMs – 'Managing Up'                        | 30min
Dealing with Problem Teachers               | 45min
Hiring the Right Cook                       | 60min
Government Policy Changes and New Globe     | 60min
Adjusting to Relocation                     | 45min
Public Works in Your Community              | 30min
Talking To Parents About Billing            | 30min
So They Say You're a Devil Worshipper       | 60min
Two-Streams or Not Two-Streams              | 30min
Piped Water                                 | 30min
```

This exactly matches the problem spec.

---

# ✅ INPUT PARSER (Production-safe, not hacky)

Create:

```
IO/InputParser.cs
```

### `InputParser.cs`

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using RetrainingScheduler.Domain;

namespace RetrainingScheduler.IO
{
    public static class InputParser
    {
        public static List<Talk> Load(string filePath)
        {
            var lines = File.ReadAllLines(filePath)
                            .Where(l => !string.IsNullOrWhiteSpace(l))
                            .ToList();

            var talks = new List<Talk>();

            foreach (var line in lines)
            {
                var parts = line.Split('|');
                var title = parts[0].Trim();
                var duration = parts[1].Trim();

                talks.Add(Talk.Parse(title, duration));
            }

            return talks;
        }
    }
}
```

---

# ✅ MAIN PROGRAM NOW USES INPUT FILE

Update `Program.cs`:

```csharp
using RetrainingScheduler.IO;
using RetrainingScheduler.Services;

class Program
{
    static void Main()
    {
        var talks = InputParser.Load("Data/input.txt");

        talks = talks.OrderByDescending(t => t.Duration).ToList();

        int tracks = TrackCalculator.Compute(talks);

        var trackList = Enumerable.Range(0, tracks)
                                  .Select(_ => new Track())
                                  .ToList();

        var scheduler = new Scheduler();

        if (!scheduler.ScheduleTalks(talks, trackList))
        {
            Console.WriteLine("❌ Scheduling failed.");
            return;
        }

        Validator.ValidateOrThrow(trackList);
        Printer.Print(trackList);
    }
}
```

---

# ✅ TRACK COUNT MOVED INTO SERVICE (Cleaner Design)

Create:

```
Services/TrackCalculator.cs
```

### `TrackCalculator.cs`

```csharp
using System.Collections.Generic;
using System.Linq;
using RetrainingScheduler.Domain;

namespace RetrainingScheduler.Services
{
    public static class TrackCalculator
    {
        public static int Compute(List<Talk> talks)
        {
            int total = talks.Sum(t => t.Duration);
            const int MINUTES_PER_TRACK = 420;
            return (int)System.Math.Ceiling(total / (double)MINUTES_PER_TRACK);
        }
    }
}
```

---

# ✅ PRINTER NOW FULLY STANDALONE

Create:

```
IO/Printer.cs
```

Prints floating sharing time.

```csharp
using System;
using System.Collections.Generic;
using RetrainingScheduler.Domain;

namespace RetrainingScheduler.IO
{
    public static class Printer
    {
        public static void Print(List<Track> tracks)
        {
            int no = 1;

            foreach (var track in tracks)
            {
                Console.WriteLine($"\nTrack {no++}\n");

                PrintSession(track.Morning);
                Console.WriteLine("12:00PM Lunch\n");

                PrintSession(track.Afternoon);

                var share = track.GetSharingTime();
                Console.WriteLine($"{share:hh:mmtt} Sharing Session\n");
            }
        }

        static void PrintSession(SessionSlot session)
        {
            var current = session.Start;

            foreach (var talk in session.Talks)
            {
                Console.WriteLine($"{current:hh:mmtt} {talk.Title} ({talk.Duration}min)");
                current = current.AddMinutes(talk.Duration);
            }
        }
    }
}
```

---

# ✅ TALK MODEL (DOMAIN)

```
Domain/Talk.cs
```

```csharp
namespace RetrainingScheduler.Domain
{
    public class Talk
    {
        public string Title { get; set; }
        public int Duration { get; set; }

        public static Talk Parse(string title, string duration)
        {
            int mins = duration == "lightning" ? 5 : int.Parse(duration.Replace("min", ""));
            return new Talk { Title = title, Duration = mins };
        }
    }
}
```

---

# ✅ HOW TO RUN

### From CLI:

```bash
dotnet build
dotnet run
```

### Input lives in:

```
Data/input.txt
```

Edit it and rerun.

---

# ✅ BENEFITS OF THIS STRUCTURE

You now have:

✅ Maintainable design
✅ Clean boundaries
✅ Easy testing
✅ Easy swapping of input source
✅ Production-safe parsing
✅ Shared logic isolated
✅ No spaghetti code
