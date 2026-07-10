# may or may not be good cause it took me 4 tries to get ts workin
	TAS Tool v4 — Idle / Record / Test, frame-stepping, branch-on-rewind,
	respawn-safe, and death-rescue
	
	Controls:
		Left Click  -> toggle Pause / Unpause (freezes your character in place)
		1           -> Idle mode      (free control, nothing recorded)
		2           -> Recording mode (captures your position + health every frame)
		3           -> Testing mode   (replays the recording on your character)
 
		F -> step back one frame
		G -> step forward one frame
		R -> hold to rewind continuously, stops at frame 0
		T -> hold to fast-forward continuously, stops at the last recorded frame
		(F/G/R/T work in Testing mode any time, and in Recording mode while paused)
 
	Rewind-and-overwrite (Recording mode):
		Pause mid-recording, rewind a few frames with F/R, then unpause. Everything
		AFTER the frame you rewound to gets discarded, and new frames get appended
		from there instead — you just branched a new timeline off the old one.
 
	Death rescue:
		Every recorded frame stores your Health along with your CFrame. The
		instant Humanoid.HealthChanged reports 0 (or less), the script:
			1. immediately restores Health to whatever it was at your current frame
			2. snaps your CFrame back to that same frame
			3. pauses, so you don't keep recording/playing through a death
		This intercepts BEFORE Roblox's death sequence actually plays out — it's
		the standard "catch the health drop before it registers as dead" trick,
		plus BreakJointsOnDeath is disabled as a backup so you don't ragdoll even
		if something slips through. It's a best-effort catch, not a guarantee:
		it won't save you from something that destroys the character directly
		instead of damaging it through Health (e.g. a script calling
		character:Destroy() rather than Humanoid:TakeDamage()). For a toolbox
		rig you're just messing around with for friends, this covers it.
 
	Respawn safety (fallback, in case death rescue somehow doesn't catch it):
		Re-hooks to the new character, reapplies lock state, repositions to the
		current frame if in Testing mode, and re-registers the death-rescue hook
		on the fresh Humanoid.
 
	"Frame" = one RunService.Heartbeat tick. Recording stores one entry per
	Heartbeat while unpaused; Testing mode plays back one frame per Heartbeat,
	so it lines up with roughly the same real-world speed it was recorded at.
