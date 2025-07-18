# Bracketeer - for fun and glory.

A work-in-progress tool to run the timers for matches for combat robotics, as well as some stream overlay stuff.

This was initially built for the [Garden State Combat Robotics League](https://www.gscrl.org) but much of it is portable to other organizing bodies.

## Core Features

- **Multi-Tournament Management**: View brackets from multiple tournaments in one unified interface
- **Real-Time Match Timers**: Precision timing with emergency stop, pause/resume functionality
- **Stream Overlays**: Professional graphics for live streaming and broadcast
- **Multi-Arena Support**: Coordinate simultaneous matches across multiple arenas
- **Fight Log**: Comprehensive record of all matches per tournament with robot statistics
- **Match Queue Management**: "On deck" system showing upcoming matches
- **TrueFinals Integration**: Direct API integration with automatic match state polling
- **Competitor Displays**: Red/blue corner displays showing match info and timer
- **SocketIO Real-Time Updates**: All displays update instantly via WebSocket communication

![A screenshot of the upcoming matches screen showing multiple matches and some active.](./_repo/upcoming.png)

![A screenshow showing the control pane for match timer use, as well as the individual robot loading dialogue to apply names to ensure competitors are in the right place.](./_repo/match_control.png)

## Installation

This tool requires Python 3.9+ and `uv` for dependency management.

### Quick Start

1. **Install Python 3.9+** and [uv](https://docs.astral.sh/uv/)
2. **Clone the repository** and navigate to the project directory
3. **Install dependencies:**
   ```bash
   uv sync --group dev
   uv pip install -e .
   ```
4. **Run the application:**
   ```bash
   # Development mode (recommended for testing)
   ./scripts/start-development.sh
   
   # Or manually
   python -m bracketeer --dev
   ```

### Development Server Management

**Starting Development Server:**
```bash
# Automated start with auto-reload
./scripts/start-development.sh

# Manual start with custom options
python -m bracketeer --dev --port 8080 --debug
python -m bracketeer --dev --host 127.0.0.1 --port 5000
```

**Stopping Development Server:**
```bash
# Keyboard interrupt (development servers run in foreground)
Ctrl+C
```

**Development vs Production:**
- **Development**: Auto-reload, debug mode, detailed error pages, port auto-detection
- **Production**: Optimized performance, production logging, manual port configuration

### Production Deployment

**Starting Production Server:**
```bash
# Automated start with script
./scripts/start-production.sh

# Manual start
export BRACKETEER_ENV=production
python -m bracketeer --host 0.0.0.0 --port 80 --no-debug
```

**Stopping Production Server:**
```bash
# Method 1: Keyboard interrupt (if running in foreground)
Ctrl+C

# Method 2: Find and kill process (if running in background)
ps aux | grep bracketeer
kill <process_id>

# Method 3: Kill by port (if needed)
sudo lsof -ti:80 | xargs kill -9
```

**Background Operation:**
```bash
# Run in background (detached from terminal)
nohup ./scripts/start-production.sh > logs/production.log 2>&1 &

# View logs
tail -f logs/production.log

# Stop background process
ps aux | grep bracketeer
kill <process_id>
```

## Networking Setup

When running the host computer, setting a static IP address is not optional.  If not done so, you may have timer clients disconnect and unable to find the origin.

GSCRL uses `192.168.8.250` due to the DHCP range by default of our [travel router](https://www.amazon.com/GL-iNet-GL-SFT1200-Secure-Travel-Router/dp/B09N72FMH5).  Anything will work, but be consistent and set the netmask properly (/24 | 255.255.255.0 by default)

GSCRL *also* sets a DNS override in the DNS server on the travel router to make `192.168.8.250` point to `arena.gscrl.org`.  This was a legacy holdover from when using slow-rotating but externally validated Let's Encrypt setups even offline, but was retained for convenience.

## Testing on One Computer

You can test the entire Bracketeer system on a single laptop by opening multiple browser windows/tabs to simulate all user roles:

### Quick Test Setup

1. **Start the application**: `uv run bracketeer/__main__.py`
2. **Open these URLs in separate browser windows**:
   - **Judge Control**: `http://localhost/control/1`
   - **Red Competitor**: `http://localhost/screens/1/timer/red` 
   - **Blue Competitor**: `http://localhost/screens/1/timer/blue`
   - **Stream Overlay**: `http://localhost/screens/1/timer`
   - **Match Queue**: `http://localhost/matches/upcoming`
   - **Main Dashboard**: `http://localhost/`

3. **Arrange windows** on your screen to see all interfaces simultaneously
4. **Test the workflow**:
   - Click "MARK READY" in red/blue competitor windows
   - Use judge control to start/pause/stop timers
   - Watch real-time updates across all displays
   - Test emergency stop functionality

### Key Features in Detail

**Match Queue Management:**
- **Upcoming Matches**: `/matches/upcoming` - Shows next matches to be called
- **"On Deck" System**: Matches marked as `"called"` appear prominently 
- **Tournament Context**: Each match clearly shows which tournament it belongs to
- **Real-Time Updates**: Automatic polling every 15 seconds from TrueFinals API
- **Match Progression**: `"available"` → `"called"` (on deck) → `"active"` (fighting) → `"done"`

**Fight Log System:**
- **Complete Match History**: `/matches/fight-log` - All fights per tournament
- **Robot Records**: Win/loss statistics for each competitor
- **Tournament Filtering**: Filter by specific tournament or view all
- **Match Organization**: Separated by completed, in-progress, and upcoming
- **Progress Tracking**: Visual progress bars showing tournament completion

**Timer Control System:**
- **Judge Interface**: `/control/{cageID}` - Full timer control with emergency stop
- **Competitor Displays**: Separate red/blue corner screens with ready indicators
- **Stream Overlay**: Professional timer display for broadcasts
- **SocketIO Coordination**: All displays update simultaneously in real-time

All displays update in real-time via SocketIO, making single-computer testing effective for development and demonstration.

## Server Management & Troubleshooting

### Common Issues

**Port Already in Use:**
```bash
# Check what's using port 80
sudo lsof -i :80

# Kill process using port 80
sudo lsof -ti:80 | xargs kill -9

# Or use a different port
python -m bracketeer --port 8080 --host 0.0.0.0 --no-debug
```

**Permission Denied (Port 80):**
```bash
# Option 1: Run with sudo (not recommended for development)
sudo python -m bracketeer --host 0.0.0.0 --port 80 --no-debug

# Option 2: Use a higher port (recommended)
python -m bracketeer --host 0.0.0.0 --port 8080 --no-debug
```

**Server Won't Stop:**
```bash
# Find all Bracketeer processes
ps aux | grep bracketeer | grep -v grep

# Kill all Bracketeer processes
pkill -f "python.*bracketeer"

# Nuclear option: kill all Python processes using port 80
sudo lsof -ti:80 | xargs kill -9
```

**Check if Server is Running:**
```bash
# Test if server responds
curl http://localhost:80
curl http://192.168.8.250:80

# Check process list
ps aux | grep bracketeer
```

### Production Server Status

**View Server Logs:**
```bash
# If running with nohup
tail -f logs/production.log

# If running with systemd (advanced)
journalctl -u bracketeer -f
```

**Monitor Server Health:**
```bash
# Check if server is responding
while true; do curl -s http://localhost:80 > /dev/null && echo "$(date): Server OK" || echo "$(date): Server DOWN"; sleep 10; done
```

## Additional Tools

The `_notebooks` directory contains any tools built to make running a league easier (such as the GSCRL District Point model for advancement to the season championship).

To run said notebooks, install jupyter and then run `jupyter notebook`.