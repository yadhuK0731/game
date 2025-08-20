# Supabase Setup Instructions

## Step 1: Create Supabase Project

1. Go to [supabase.com](https://supabase.com)
2. Sign up or log in
3. Click "New Project"
4. Fill in project details:
   - Name: moo-games (or your preferred name)
   - Database Password: (choose a strong password and save it)
   - Region: Choose closest to your users
5. Click "Create new project"

## Step 2: Set up Database

1. Wait for project creation to complete
2. Go to SQL Editor in your Supabase dashboard
3. Copy and paste the following SQL:

```sql
-- Create players table to store game progress
CREATE TABLE players (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  player_name VARCHAR(20) UNIQUE NOT NULL,
  milk BIGINT DEFAULT 0,
  cows INTEGER DEFAULT 1,
  milk_per_click INTEGER DEFAULT 1,
  milk_per_second INTEGER DEFAULT 0,
  milking_upgrades INTEGER DEFAULT 0,
  cow_upgrades INTEGER DEFAULT 0,
  farm_upgrades INTEGER DEFAULT 0,
  milking_cost INTEGER DEFAULT 10,
  cow_cost INTEGER DEFAULT 50,
  farm_cost INTEGER DEFAULT 200,
  total_milk_collected BIGINT DEFAULT 0,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Create an index on player_name for faster lookups
CREATE INDEX idx_players_name ON players(player_name);

-- Create a function to update the updated_at timestamp
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ language 'plpgsql';

-- Create a trigger to automatically update updated_at
CREATE TRIGGER update_players_updated_at
    BEFORE UPDATE ON players
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at_column();

-- Enable Row Level Security (RLS)
ALTER TABLE players ENABLE ROW LEVEL SECURITY;

-- Create policy to allow anyone to read and write
-- Note: In production, you might want more restrictive policies
CREATE POLICY "Allow all operations on players" ON players
    FOR ALL USING (true) WITH CHECK (true);
```

4. Click "Run" to execute the SQL

## Step 3: Get Credentials

1. Go to Project Settings → API
2. Copy the following values:
   - Project URL
   - `anon` `public` key

## Step 4: Update Your Game

1. Open `index.html`
2. Find these lines near the top of the script section:

```javascript
const SUPABASE_URL = "YOUR_SUPABASE_URL";
const SUPABASE_ANON_KEY = "YOUR_SUPABASE_ANON_KEY";
```

3. Replace with your actual values:

```javascript
const SUPABASE_URL = "https://your-project-ref.supabase.co";
const SUPABASE_ANON_KEY = "your-anon-key-here";
```

## Step 5: Test

1. Open your game in a browser
2. Enter a player name
3. Play the Cow Farm Empire game
4. Check your Supabase dashboard → Table Editor → players to see if data is being saved

## Features Implemented

- ✅ Player name-based authentication (simple login)
- ✅ Automatic save/load of Cow Farm Empire progress
- ✅ Auto-save every 30 seconds
- ✅ Save on page unload
- ✅ Total milk collected tracking
- ✅ All farm upgrades and costs preserved

## Database Schema

The `players` table stores:

- `player_name`: Unique player identifier (max 20 chars)
- `milk`: Current milk amount
- `cows`: Number of cows owned
- `milk_per_click`: Milk gained per click
- `milk_per_second`: Passive milk generation
- `milking_upgrades`: Number of milking upgrades purchased
- `cow_upgrades`: Number of cow upgrades purchased
- `farm_upgrades`: Number of farm upgrades purchased
- `milking_cost`, `cow_cost`, `farm_cost`: Current upgrade costs
- `total_milk_collected`: Lifetime milk collected
- `created_at`, `updated_at`: Timestamps

## Security Notes

- Current setup allows public read/write access for simplicity
- For production, consider implementing proper user authentication
- Row Level Security (RLS) is enabled but with permissive policies

## Future Enhancements

- Add leaderboards
- Implement proper high score system
- Add achievements system
- Store other mini-game statistics
