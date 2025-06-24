
alias opt_ball_location_visible = script_option[0]

alias announced_game_start = player.number[0]
alias announce_start_timer = player.timer[2]

alias ball_count         = global.number[2]
alias spawn_loc_count    = global.number[3]
alias ball_spawn_timeout = global.timer[0] -- we'll loosen restrictions on ball spawn locations if this much time passes
alias has_ball        = object.number[0] -- for spawn points
alias average_distance_to_winners = object.number[2] -- leftover from Stockpile's fair spawning code
alias spawned_from    = object.object[0] -- for balls; matches them to the spawn point that created them
alias expire_timer    = object.timer[0] -- balls are deleted when this hits zero
alias noise_timer     = object.timer[1] -- for "noisemakers" and the skulls they spawn

declare global.number[0] with network priority local -- unused (set in one place; never read)
declare global.number[1] with network priority low   -- unused
declare ball_count      with network priority low
declare spawn_loc_count with network priority low
declare global.number[4] with network priority local -- temporary
declare global.number[5] with network priority local -- temporary
declare global.object[0] with network priority local -- temporary
declare global.object[1] with network priority local -- temporary
declare global.object[2] with network priority local -- temporary
declare global.team[0]   with network priority local -- temporary
declare ball_spawn_timeout = 1 -- this is a short timeout but bear in mind, when this is running we're trying to spawn a ball every frame
declare player.number[0] with network priority low
declare player.timer[1] = 1 -- unused
declare player.timer[2] = 5 -- unused
declare object.has_ball  with network priority low
declare object.number[1] with network priority low -- unused
declare object.average_distance_to_winners with network priority low
declare object.spawned_from with network priority low
declare object.player[0] with network priority low -- unused
declare object.player[1] with network priority low -- unused
declare object.expire_timer = 62
declare team.number[0] with network priority low   -- unused
declare team.object[0] with network priority low   -- unused
declare team.object[1] with network priority local -- unused
declare team.object[2] with network priority local -- unused

for each player do -- loadout palettes
   if current_player.is_elite() then 
      current_player.set_loadout_palette(elite_tier_1)
   end
   if not current_player.is_elite() then 
      current_player.set_loadout_palette(spartan_tier_1)
   end
end

for each player do -- round card and announce game start
   current_player.announce_start_timer.set_rate(-100%)
   current_player.biped.set_waypoint_icon(none)
   if game.score_to_win != 0 then 
      current_player.set_round_card_title("Score balls in the cups to earn points! \n%n to win!", game.score_to_win)
   end
   if game.score_to_win == 0 then 
      current_player.set_round_card_title("Score golf-balls in the cups to earn points")
   end
   if current_player.announced_game_start == 0 and current_player.announce_start_timer.is_zero() then 
      send_incident(action_sack_game_start, current_player, no_player)
      current_player.announced_game_start = 1
   end
end

for each object with label "hb_goal" do
   global.number[4] = current_object.spawn_sequence
   current_object.set_waypoint_visibility(allies)
   current_object.set_waypoint_icon(territory_a, global.number[4])
   current_object.set_waypoint_priority(high)
end

for each object with label "ball_spawn_loc" do
   current_object.number[1] = 0 -- unused
   current_object.has_ball  = 0
   current_object.average_distance_to_winners = 0
   global.number[4] = 0 -- this is a temporary, so not sure why it's zeroed out here
   global.number[5] = 0 -- this is a temporary, so not sure why it's zeroed out here
end

do
   spawn_loc_count = 0
   for each object with label "ball_spawn_loc" do
      spawn_loc_count += 1
   end
   ball_count = 0
   for each object with label "halo_ball" do
      ball_count += 1
   end
end

for each object with label "halo_ball" do -- manage scoring and ball lifespans
   alias current_ball        = global.object[0]
   alias total_points_scored = global.number[4] -- for all goals, if multiple goals per frame
   --
   current_ball = current_object
   current_ball.expire_timer.set_rate(-100%)
   total_points_scored = 0
   if opt_ball_location_visible == 1 then 
      current_ball.set_waypoint_visibility(everyone)
      current_ball.set_waypoint_icon(bullseye)
   end
   for each object with label "hb_goal" do
      alias goal_owner = global.team[0]
      --
      goal_owner = current_object.team
      if goal_owner.has_any_players() and current_object.shape_contains(current_ball) then
         alias points_scored = global.number[5] -- for this individual goal
         --
         current_ball.timer[0].reset()
         points_scored = current_object.spawn_sequence
         if points_scored <= 0 then 
            points_scored = 1
         end
         current_object.team.score += points_scored
         total_points_scored       += points_scored
         game.show_message_to(current_object.team, boneyard_generator_power_down, "Points Scored: %n", points_scored)
         for each object with label "goal_noise_maker" do -- fire off any noisemakers keyed to this goal
            if current_object.spawn_sequence == points_scored and current_object.team == goal_owner then 
               current_object.noise_timer = 2 -- object will make noise until the timer hits zero
            end
         end
         global.number[0] = 1 -- this variable is not checked anywhere
         current_ball.delete()
         ball_count -= 1
      end
   end
   if not current_ball == no_object then 
      for each object with label "ball_spawn_loc" do
         if current_object.shape_contains(current_ball) then 
            current_object.has_ball = 1 -- checked by ball-spawning code below
            current_ball.expire_timer.reset() -- balls don't expire if they're near their spawn
         end
      end
      do -- high-speed movement should prolong a ball's lifespan
         global.number[5] = 0
         global.number[5] = current_ball.get_speed()
         if global.number[5] > 5 then 
            current_ball.expire_timer.reset() -- moving at high speed prolongs a ball's lifespan
         end
      end
      if current_object.is_out_of_bounds() or current_object.expire_timer.is_zero() then 
         ball_count -= 1
         current_object.delete()
      end
   end
end

do -- spawn balls as needed
   --
   -- This code appears to have been adapted from Stockpile.
   --
   ball_spawn_timeout.set_rate(100%)
   if ball_count < spawn_loc_count then
      alias selected_spawn = global.object[0]
      alias fallback_spawn = global.object[1] -- used if we can't find an ideal spawn
      alias spawn_allowed  = global.number[4]
      --
      spawn_allowed  = 1
      selected_spawn = no_object
      selected_spawn = get_random_object("ball_spawn_loc", no_object)
      for each object with label "hb_goal" do -- don't use a spawn point if it's inside of a goal
         global.object[1] = current_object
         if global.object[1].shape_contains(selected_spawn) then 
            spawn_allowed = 0
         end
      end
      if selected_spawn.has_ball == 1 then -- don't use a spawn point if it contains any ball
         spawn_allowed = 0
      end
      for each object with label "halo_ball" do -- don't use a spawn point if a ball it created still exists
         if current_object.spawned_from == selected_spawn then 
            spawn_allowed = 0
         end
      end
      if spawn_allowed == 1 or ball_spawn_timeout.is_zero() then 
         fallback_spawn = no_object
         for each object with label "ball_spawn_loc" do
            if  fallback_spawn.average_distance_to_winners < current_object.average_distance_to_winners -- Stockpile leftover; never set to anything meaningful
            or  fallback_spawn == no_object
            and current_object.has_ball == 0
            then 
               fallback_spawn = current_object
            end
         end
         global.object[2] = no_object
         if ball_spawn_timeout.is_zero() then 
            selected_spawn = fallback_spawn
         end
         global.object[2] = selected_spawn.place_at_me(golf_ball, "halo_ball", never_garbage_collect, 0, 0, 3, none)
         global.object[2].spawned_from = selected_spawn
      end
   end
end

if game.round_time_limit > 0 and game.round_timer.is_zero() then -- round timer
   game.end_round()
end

-- Noisemaker functionality:
for each object with label "goal_noise_maker" do -- make noise
   alias created_noise = global.object[0]
   --
   current_object.noise_timer.set_rate(-100%)
   if not current_object.noise_timer.is_zero() then 
      created_noise = no_object
      global.number[4] = 0
      global.number[4] = rand(4)
      if global.number[4] <= 1 then 
         created_noise = current_object.place_at_me(skull, "goal_noise", never_garbage_collect, 0, 0, 0, none)
      end
      if created_noise != no_object then 
         created_noise.push_upward()
         created_noise.set_pickup_permissions(no_one)
         created_noise.noise_timer = 1
      end
   end
end
for each object with label "goal_noise" do -- noise lifespan
   current_object.noise_timer.set_rate(-100%)
   if current_object.noise_timer.is_zero() then
      --
      -- We initialize the timer to 0 and manually assign 1 to it; this means 
      -- that pre-placed Forge objects with the GOAL_NOISE label always have 
      -- the timer set to zero and are deleted instantly.
      --
      current_object.delete()
   end
end
