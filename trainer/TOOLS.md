# TOOLS.md - Coach Chaos Reference

## Workout Tracker API

Base URL: `https://workout.emilycogsdill.com/api`
Auth: `Authorization: Bearer $WORKOUT_API_KEY`

### Key Endpoints

```bash
# All workouts (most recent first)
curl -H "Authorization: Bearer $WORKOUT_API_KEY" \
  "https://workout.emilycogsdill.com/api/workouts"

# All personal records
curl -H "Authorization: Bearer $WORKOUT_API_KEY" \
  "https://workout.emilycogsdill.com/api/workouts/prs/all"

# PRs for specific exercise (URL-encode name)
curl -H "Authorization: Bearer $WORKOUT_API_KEY" \
  "https://workout.emilycogsdill.com/api/workouts/prs/Bench%20Press"

# Exercise definitions
curl -H "Authorization: Bearer $WORKOUT_API_KEY" \
  "https://workout.emilycogsdill.com/api/exercises"
```

## Data Structures

### Workout
```json
{
  "id": "uuid",
  "start_time": 1706900000000,
  "end_time": 1706903600000,
  "target_categories": ["Chest", "Triceps"],
  "exercises": [
    {
      "name": "Bench Press",
      "sets": [
        {"weight": 135, "reps": 10, "isPR": false, "completed": true},
        {"weight": 155, "reps": 8, "isPR": true, "completed": true}
      ]
    }
  ]
}
```

### Categories
Chest, Shoulders, Triceps, Back, Biceps, Legs, Core, Cardio, Other

### Weight Types
- `+bar` — Add 45 lbs for standard barbell
- `total` — Log total weight shown
- `/side` — Weight per dumbbell
- `bodyweight` — No added weight

## Training Principles

### Progressive Overload
- Add 5 lbs when all target reps completed
- Add reps before adding weight
- Deload 10% if stuck for 2+ sessions

### Volume Guidelines
- 10-20 sets per muscle group per week
- Spread across 2-3 sessions
- More isn't always better

### Recovery
- 48-72 hours between same muscle group
- Sleep and nutrition matter more than any program
- Deload week every 4-6 weeks

## Standard Rep Schemes

| Goal | Sets | Reps | Rest |
|------|------|------|------|
| Strength | 4-5 | 3-5 | 3-5 min |
| Hypertrophy | 3-4 | 8-12 | 60-90 sec |
| Endurance | 2-3 | 15-20 | 30-60 sec |
