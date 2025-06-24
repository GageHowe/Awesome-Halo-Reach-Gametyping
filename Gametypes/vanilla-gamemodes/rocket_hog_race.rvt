
alias opt_landmines     = script_option[0]
alias opt_teleport_time = script_option[1]
alias opt_hill_order    = script_option[2]
alias opt_vehicle       = script_option[3]
enum race_vehicles
   mongoose = 1
   warthog  = 2
end

alias vehicle_traits = script_traits[0]
alias on_foot_traits = script_traits[1]
alias boarder_traits = script_traits[2]

-- Unnamed Forge labels:
alias all_landmines  = 3
alias all_weak_antis = 5

alias top_speed         = player.script_stat[0]
alias distance_traveled = player.script_stat[1]
alias rotations         = player.script_stat[2]

alias announced_game_start = player.number[6]
alias announce_start_timer = player.timer[0]

alias is_after_initial  = global.number[0] -- used to avoid playing "Destination moved" for the first goal
alias temp_int_00       = global.number[1]
alias temp_int_01       = global.number[2]
alias temp_int_02       = global.number[3]
alias current_goal      = global.object[0]
alias previous_goal     = global.object[1]
alias movement_stats_update_interval = global.timer[0]
alias goal_move_timeout = global.timer[1]
alias distance_to_prev  = object.number[0] -- distance from this checkpoint to the previous one
alias cannot_be_next    = object.number[1] -- some condition is met such that this goal can't be the next one right now
alias last_orientation  = object.number[2]
alias turns_90deg_pitch = object.number[3]
alias turns_90deg_roll  = object.number[4]
alias turns_90deg_both  = object.number[5]
alias mine_is_armed     = object.number[6]
alias detonation_timer  = object.timer[2]
alias role = player.number[0]
enum role
   undefined  = -1
   on_foot    = 3
   in_vehicle = 4
end
alias speed            = player.number[1]
alias speed_kph        = player.number[2]
alias distance_to_goal = player.number[3] -- in meters
alias teleport_timer   = player.timer[1]
alias teleport_beeper  = player.timer[2]
alias any_in_vehicle   = team.number[0]
alias vehicle          = team.object[0]

declare is_after_initial with network priority local
declare temp_int_00      with network priority local
declare temp_int_01      with network priority local
declare temp_int_02      with network priority local
declare current_goal  with network priority low
declare previous_goal with network priority low
declare global.object[2] with network priority local
declare global.object[3] with network priority local
declare global.player[0] with network priority local
declare global.team[0] with network priority local
declare movement_stats_update_interval = 1
declare goal_move_timeout = 2
declare player.role      with network priority low = -1
declare player.speed     with network priority local
declare player.speed_kph with network priority local
declare player.distance_to_goal with network priority low
declare player.number[4] with network priority local
declare player.number[5] with network priority low
declare player.announced_game_start with network priority low
declare player.number[7] with network priority low
declare player.object[0] with network priority local
declare player.announce_start_timer = 5
declare player.teleport_timer  = opt_teleport_time
declare player.teleport_beeper = 1
declare object.distance_to_prev  with network priority low
declare object.cannot_be_next    with network priority low
declare object.last_orientation  with network priority low = 1
declare object.turns_90deg_pitch with network priority low
declare object.turns_90deg_roll  with network priority low
declare object.turns_90deg_both  with network priority low
declare object.mine_is_armed with network priority low
declare object.timer[0] = 16
declare object.detonation_timer = 2
declare team.any_in_vehicle with network priority low
declare team.number[1] with network priority low
declare team.vehicle with network priority low
declare team.player[0] with network priority low
declare team.player[1] with network priority low
declare team.timer[0] = 2

on init: do
   for each player do
      current_player.teleport_timer = 0
   end
end

for each player do -- round card and announce game start
   current_player.announce_start_timer.set_rate(-100%)
   if game.score_to_win != 0 then 
      current_player.set_round_card_title("Drive through flags to earn points.\r\n%n points to win.", game.score_to_win)
   end
   if game.score_to_win == 0 then 
      current_player.set_round_card_title("Drive through flags to earn points.")
   end
   if current_player.announced_game_start == 0 and current_player.announce_start_timer.is_zero() then 
      send_incident(rocket_race_game_start, current_player, no_player)
      current_player.announced_game_start = 1
   end
end

for each player do -- create vehicles for teams that don't have them
   alias owner_team = global.team[0]
   --
   owner_team = current_player.team
   if owner_team.vehicle == no_object then 
      global.object[2] = current_player.biped
      if not global.object[2] == no_object then 
         if opt_vehicle == race_vehicles.mongoose then 
            owner_team.vehicle = current_player.biped.place_at_me(mongoose, "sweet_ride_bro", none, 0, 0, 0, none)
         end
         if opt_vehicle == race_vehicles.warthog then 
            owner_team.vehicle = current_player.biped.place_at_me(warthog, "sweet_ride_bro", none, 0, 0, 0, rocket)
         end
         current_player.force_into_vehicle(owner_team.vehicle)
         global.object[3] = owner_team.vehicle
         global.object[3].team = owner_team
      end
   end
end

for each team do
   current_team.any_in_vehicle = 0
end
for each player do -- player traits and team.any_in_vehicle
   alias current_vehicle = global.object[2]
   alias player_team     = global.team[0]
   --
   current_vehicle = no_object
   player_team     = current_player.team
   current_vehicle = current_player.try_get_vehicle()
   if current_vehicle != no_object and current_vehicle != player_team.vehicle then 
      current_player.apply_traits(boarder_traits)
   end
   if current_vehicle != no_object then 
      current_player.apply_traits(vehicle_traits)
      current_player.role = role.in_vehicle
      player_team.any_in_vehicle = 1
   end
   if current_vehicle == no_object then 
      current_player.apply_traits(on_foot_traits)
      current_player.role = role.on_foot
   end
end

for each team do -- set up spawning
   if current_team.has_any_players() then 
      current_team.set_co_op_spawning(false)
      if current_team.vehicle != no_object then 
         current_team.set_primary_respawn_object(current_team.vehicle)
      end
   end
end

for each player do
   script_widget[0].set_visibility(current_player, false)
   global.team[0] = current_player.team
   script_widget[0].set_text("%n KPH", hud_player.speed_kph)
   current_player.distance_to_goal = 0
   current_player.distance_to_goal = current_player.biped.get_distance_to(current_goal)
   current_player.distance_to_goal *= 7
   current_player.distance_to_goal /= 23
   global.object[2] = current_player.biped
   if not global.object[2] == no_object and current_player.role == role.in_vehicle then 
      script_widget[0].set_visibility(current_player, true)
   end
end

on local: do -- track player speed for UI
   for each player do
      current_player.speed      = current_player.biped.get_speed()
      current_player.speed_kph  = current_player.speed
      current_player.speed_kph *= 109
      current_player.speed_kph /= 100 -- feet per second to kilometers per hour
   end
end

do -- update player movement stats
   movement_stats_update_interval.set_rate(-100%)
   if movement_stats_update_interval.is_zero() then 
      for each player do
         temp_int_00 = 1
         temp_int_01 = current_player.speed
         temp_int_00 *= temp_int_01
         temp_int_00 *= 10
         temp_int_00 /= 100
         current_player.distance_traveled += temp_int_00
         if current_player.speed_kph > current_player.top_speed then 
            current_player.top_speed = current_player.speed_kph
         end
      end
      movement_stats_update_interval.reset()
   end
end

for each player do
   alias player_team = global.team[0]
   alias vehicle     = global.object[2]
   alias vehicle_belongs_to_other_team = temp_int_00
   --
   player_team = current_player.team
   vehicle     = no_object
   vehicle     = current_player.try_get_vehicle()
   if not vehicle == no_object then 
      vehicle_belongs_to_other_team = 0
      for each team do
         if current_team.vehicle == vehicle and current_team != player_team then 
            vehicle_belongs_to_other_team = 1
         end
      end
      if vehicle_belongs_to_other_team == 0 then 
         player_team.vehicle = vehicle
         vehicle.team = player_team
      end
   end
end

for each team do -- vehicle abandonment (all on foot; any 50+ Forge units away)
   if current_team.has_any_players() then
      alias distance    = temp_int_00
      alias player_team = global.team[0]
      for each player do
         distance    = 0
         player_team = current_player.team
         if current_team == player_team then 
            distance = player_team.vehicle.get_distance_to(current_player.biped)
            if distance >= 500 and current_team.any_in_vehicle == 0 then 
               current_team.vehicle = no_object
            end
         end
      end
   end
end

for each player do -- vehicle abandonment (all on foot; any dead)
   if current_player.killer_type_is(guardians | suicide | kill | betrayal | quit) then 
      global.team[0] = current_player.team
      if global.team[0].any_in_vehicle == 0 then 
         global.team[0].vehicle = no_object
      end
   end
end

if current_goal == no_object then -- select next goal
   for each object with label all_weak_antis do
      current_object.delete()
   end
   current_goal = get_random_object("rally_flag", previous_goal)
   if opt_hill_order == 1 and not goal_move_timeout.is_zero() and current_goal.cannot_be_next == 1 then
      --
      -- If hill selection fails, retry, unless the timeout period has run out in which 
      -- case just settle for what the randomization picked.
      --
      current_goal = no_object
      goal_move_timeout.set_rate(-100%)
   end
   if not current_goal == no_object then 
      goal_move_timeout.reset()
      goal_move_timeout.set_rate(0%)
      current_goal.team = neutral_team
      current_goal.set_waypoint_visibility(everyone)
      current_goal.set_waypoint_icon(flag)
      current_goal.set_waypoint_text("%nm", hud_player.distance_to_goal)
      current_goal.set_shape_visibility(everyone)
      if is_after_initial == 1 then 
         game.show_message_to(all_players, announce_destination_moved, "Destination Moved.")
      end
      current_goal.set_waypoint_priority(high)
      --
      -- Discourage spawning near the goal:
      --
      global.object[2] = no_object
      global.object[2] = current_goal.place_at_me(respawn_zone_weak_anti, none, never_garbage_collect, 0, 0, 0, none)
      global.object[2].team = neutral_team
      --
      is_after_initial = 1
   end
end

for each object with label "rally_flag" do -- scoring
   current_object.set_shape_visibility(no_one)
   if current_goal == current_object then
      alias any_scored = temp_int_00
      --
      global.object[2] = current_object
      any_scored = 0
      current_object.set_shape_visibility(everyone)
      for each player do
         if any_scored == 0 and current_object.shape_contains(current_player.biped) then 
            any_scored = 1
            current_player.team.score += 1
            send_incident(checkpoint_reached_team, current_player, no_player)
         end
      end
      if any_scored == 1 then 
         previous_goal = current_goal
         current_goal  = no_object
         goal_move_timeout.reset()
         global.object[2].set_waypoint_visibility(no_one)
         global.object[2].set_shape_visibility(no_one)
      end
   end
end

for each object with label "rally_flag" do -- goal selection: don't allow goals too close to the previous goal
   if current_goal == no_object then 
      current_object.cannot_be_next = 0
      if not previous_goal == no_object then 
         current_object.distance_to_prev = previous_goal.get_distance_to(current_object)
         if current_object.distance_to_prev < 50 then 
            current_object.cannot_be_next = 1
            goal_move_timeout.set_rate(-100%)
         end
      end
   end
end

for each player do -- goal selection: don't allow goals that a player is already close to or inside
   alias distance = temp_int_00
   if current_goal == no_object then 
      global.player[0] = current_player
      distance = 0
      for each object with label "rally_flag" do
         distance = global.player[0].biped.get_distance_to(current_object)
         if current_object.shape_contains(global.player[0].biped) or distance < 30 then 
            current_object.cannot_be_next = 1
            goal_move_timeout.set_rate(-100%)
         end
      end
   end
end

if game.round_time_limit > 0 and game.round_timer.is_zero() then -- round timer
   game.end_round()
end

for each object with label "none" do
   current_object.delete()
end

for each object with label all_landmines do -- landmine handling
   alias current_landmine = global.object[2]
   --
   if opt_landmines == 0 then 
      current_object.delete()
   end
   current_object.set_waypoint_visibility(everyone)
   current_object.set_waypoint_priority(low)
   current_object.set_waypoint_range(0, 20)
   current_object.set_shape(sphere, 10)
   current_object.set_shape_visibility(everyone)
   current_landmine = current_object
   for each player do
      if current_landmine.mine_is_armed == 0 and current_landmine.shape_contains(current_player.biped) then 
         current_landmine.mine_is_armed = 1
         current_landmine.detonation_timer.set_rate(-100%)
         current_landmine.set_waypoint_priority(blink)
         current_landmine.set_waypoint_icon(bomb)
      end
   end
   if current_object.detonation_timer.is_zero() then 
      current_object.kill(false)
   end
end

for each object with label "sweet_ride_bro" do -- destroy vehicles that are too slow for too long...
   alias speed = temp_int_00
   --
   speed = 0
   speed = current_object.get_speed()
   if speed < 50 then 
      current_object.timer[0].set_rate(-100%)
      if current_object.timer[0].is_zero() then 
         for each team do
            if current_team.vehicle == current_object then 
               current_team.vehicle = no_object
            end
         end
         current_object.kill(false)
      end
   end
end
for each object with label "sweet_ride_bro" do -- ...unless they're near a player
   alias distance = temp_int_00
   --
   global.object[2] = no_object
   distance = 0
   for each player do
      distance = current_object.get_distance_to(current_player.biped)
      if distance < 10 then 
         current_object.timer[0].reset()
      end
   end
end

for each object with label "sweet_ride_bro" do
   global.team[0] = current_object.team
   if current_object != global.team[0].vehicle then 
      current_object.kill(false)
   end
end

for each team do -- destroy vehicles that belong to teams whose players have all quit
   if not current_team.has_any_players() then 
      for each object with label "sweet_ride_bro" do
         if current_object.team == current_team then 
            current_object.kill(false)
         end
      end
   end
end

for each player do -- reset teleport timer for vehicles that are killed or in their own team's vehicle
   alias vehicle = global.object[2]
   --
   vehicle = no_object
   vehicle = current_player.try_get_vehicle()
   if current_player.killer_type_is(guardians | suicide | kill | betrayal | quit) then 
      current_player.teleport_timer = 0
   end
   if vehicle != no_object and vehicle.team == current_player.team then 
      current_player.teleport_timer.reset()
   end
end

for each player do -- teleport players back into their vehicles if possible
   alias orientation = temp_int_00
   --
   global.team[0] = current_player.team
   global.object[2] = no_object
   global.object[3] = current_player.biped
   orientation = 0
   orientation = global.team[0].vehicle.get_orientation()
   global.object[2] = current_player.try_get_vehicle()
   script_widget[1].set_visibility(current_player, false)
   script_widget[3].set_visibility(current_player, false)
   if global.object[2] == no_object and global.object[3] != no_object then 
      script_widget[1].set_text("Teleporting to Vehicle: %s", hud_player.timer[1])
      script_widget[1].set_visibility(current_player, true)
      if orientation != enums.orientation.upright and global.team[0].any_in_vehicle == 0 and current_player.teleport_timer.is_zero() then 
         script_widget[3].set_text("Flip your vehicle!")
         script_widget[1].set_visibility(current_player, false)
         script_widget[3].set_visibility(current_player, true)
      end
      current_player.teleport_timer.set_rate(-100%)
      if current_player.teleport_timer.is_zero() and orientation == enums.orientation.up_is_up then 
         current_player.force_into_vehicle(global.team[0].vehicle)
      end
   end
end
for each player do -- teleport timer: beeps counting down as it nears zero
   if current_player.teleport_timer < 4 and not current_player.teleport_timer.is_zero() then 
      current_player.teleport_beeper.set_rate(-100%)
      if current_player.teleport_beeper.is_zero() then 
         game.play_sound_for(current_player, timer_beep, false)
         current_player.teleport_beeper.reset()
      end
   end
end

for each object with label "sweet_ride_bro" do
   alias owner_vehicle = global.team[0]
   alias orientation   = temp_int_00
   alias e_o = enums.orientation
   --
   orientation = 0
   owner_vehicle = current_object.team
   orientation = current_object.get_orientation()
   if current_object.last_orientation != orientation and owner_vehicle.any_in_vehicle == 1 then 
      do
         if current_object.last_orientation == e_o.up_is_up and orientation == e_o.backward_is_up then 
            current_object.turns_90deg_pitch += 1
         end
         if current_object.last_orientation == e_o.backward_is_up and orientation == e_o.down_is_up then 
            current_object.turns_90deg_pitch += 1
         end
         if current_object.last_orientation == e_o.down_is_up and orientation == e_o.forward_is_up then 
            current_object.turns_90deg_pitch += 1
         end
         if current_object.last_orientation == e_o.forward_is_up and orientation == e_o.up_is_up then 
            current_object.turns_90deg_pitch += 1
         end
      end
      do
         if current_object.last_orientation == e_o.backward_is_up and orientation == e_o.up_is_up then 
            current_object.turns_90deg_pitch += 1
         end
         if current_object.last_orientation == e_o.down_is_up and orientation == e_o.backward_is_up then 
            current_object.turns_90deg_pitch += 1
         end
         if current_object.last_orientation == e_o.forward_is_up and orientation == e_o.down_is_up then 
            current_object.turns_90deg_pitch += 1
         end
         if current_object.last_orientation == e_o.up_is_up and orientation == e_o.forward_is_up then 
            current_object.turns_90deg_pitch += 1
         end
      end
      do
         if current_object.last_orientation == e_o.up_is_up and orientation == e_o.right_is_up then 
            current_object.turns_90deg_roll += 1
         end
         if current_object.last_orientation == e_o.right_is_up and orientation == e_o.down_is_up then 
            current_object.turns_90deg_roll += 1
         end
         if current_object.last_orientation == e_o.down_is_up and orientation == e_o.left_is_up then 
            current_object.turns_90deg_roll += 1
         end
         if current_object.last_orientation == e_o.left_is_up and orientation == e_o.up_is_up then 
            current_object.turns_90deg_roll += 1
         end
      end
      do
         if current_object.last_orientation == e_o.right_is_up and orientation == e_o.up_is_up then 
            current_object.turns_90deg_roll += 1
         end
         if current_object.last_orientation == e_o.down_is_up and orientation == e_o.right_is_up then 
            current_object.turns_90deg_roll += 1
         end
         if current_object.last_orientation == e_o.left_is_up and orientation == e_o.down_is_up then 
            current_object.turns_90deg_roll += 1
         end
         if current_object.last_orientation == e_o.up_is_up and orientation == e_o.left_is_up then 
            current_object.turns_90deg_roll += 1
         end
      end
      do
         if current_object.last_orientation == e_o.left_is_up and orientation == e_o.backward_is_up then 
            current_object.turns_90deg_both += 1
         end
         if current_object.last_orientation == e_o.backward_is_up and orientation == e_o.right_is_up then 
            current_object.turns_90deg_both += 1
         end
         if current_object.last_orientation == e_o.right_is_up and orientation == e_o.forward_is_up then 
            current_object.turns_90deg_both += 1
         end
         if current_object.last_orientation == e_o.forward_is_up and orientation == e_o.left_is_up then 
            current_object.turns_90deg_both += 1
         end
      end
      do
         if current_object.last_orientation == e_o.backward_is_up and orientation == e_o.left_is_up then 
            current_object.turns_90deg_both += 1
         end
         if current_object.last_orientation == e_o.right_is_up and orientation == e_o.backward_is_up then 
            current_object.turns_90deg_both += 1
         end
         if current_object.last_orientation == e_o.forward_is_up and orientation == e_o.right_is_up then 
            current_object.turns_90deg_both += 1
         end
         if current_object.last_orientation == e_o.left_is_up and orientation == e_o.forward_is_up then 
            current_object.turns_90deg_both += 1
         end
      end
      current_object.last_orientation = orientation
      current_object.teleport_timer = 1
      current_object.teleport_timer.set_rate(-100%)
   end
end

for each object with label "sweet_ride_bro" do
   alias owner_team = global.team[0]
   --
   owner_team = current_object.team
   if current_object.timer[1].is_zero() then 
      temp_int_00  = current_object.turns_90deg_pitch
      temp_int_01  = current_object.turns_90deg_roll
      temp_int_02  = current_object.turns_90deg_both
      temp_int_00 += temp_int_01
      temp_int_00 += temp_int_02
      temp_int_00 /= 4
      owner_team.timer[0].reset()
      owner_team.timer[0].set_rate(-100%)
      for each player do
         if current_player.team == owner_team then 
            current_player.number[7]  = temp_int_00
            current_player.rotations += temp_int_00
         end
      end
      current_object.turns_90deg_pitch = 0
      current_object.turns_90deg_roll  = 0
      current_object.turns_90deg_both  = 0
      current_object.timer[1] = 1
      current_object.timer[1].set_rate(0%)
   end
end

for each player do
   global.team[0] = current_player.team
   script_widget[2].set_visibility(current_player, false)
   script_widget[2].set_text("Rotations Completed: %n", hud_player.number[7])
   if global.team[0].timer[0].is_zero() then 
      current_player.number[7] = 0
   end
   if not global.team[0].timer[0].is_zero() and current_player.number[7] != 0 then 
      script_widget[2].set_visibility(current_player, true)
   end
end

for each player do -- award Offensive Driver achievement
   alias killer = global.player[0]
   --
   killer = no_player
   if current_player.killer_type_is(kill) then 
      killer = current_player.try_get_killer()
      if killer != no_player then 
         send_incident(dlc_achieve_10, killer, current_player)
      end
   end
end
