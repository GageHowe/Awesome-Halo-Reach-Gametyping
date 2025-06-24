
alias opt_landmines = script_option[0]

-- Unnamed Forge labels:
alias all_landmines  = 3
alias all_anti_zones = 4
alias all_mongooses  = 6

alias ui_speed = script_widget[1]

alias top_speed         = player.script_stat[0]
alias distance_traveled = player.script_stat[1]

alias announced_game_start = player.number[7]
alias announce_start_timer = player.timer[0]

alias temp_int_00      = global.number[1] -- unused
alias temp_int_01      = global.number[2] -- unused
alias is_after_initial = global.number[3]
alias temp_int_02      = global.number[4]
alias temp_int_03      = global.number[5]
alias current_goal     = global.object[0]
alias previous_goal    = global.object[1]
alias temp_obj_00      = global.object[2]
alias temp_obj_01      = global.object[3]
alias temp_plr_00      = global.player[0]
alias movement_stats_update_interval = global.timer[0]
alias distance_to_prev = object.number[0] -- distance from this goal to the previous goal
alias goal_disallowed  = object.number[1] -- this object cannot be the next goal
alias mine_is_armed    = object.number[2]
alias expiration_timer = object.timer[0] -- vehicles are deleted after being abandoned for this long
alias detonation_timer = object.timer[1]
alias speed            = player.number[3]
alias speed_kph        = player.number[4]
alias distance_to_goal = player.number[5]
alias vehicle          = player.object[1]

declare global.number[0] with network priority low   -- unused
declare temp_int_00      with network priority local -- unused
declare temp_int_01      with network priority local -- unused
declare is_after_initial with network priority local
declare temp_int_02      with network priority local
declare temp_int_03      with network priority local
declare current_goal     with network priority low
declare previous_goal    with network priority low
declare temp_obj_00      with network priority local
declare temp_obj_01      with network priority local
declare temp_plr_00      with network priority local
declare movement_stats_update_interval = 1
declare player.number[0] with network priority low -- unused
declare player.number[1] with network priority low -- unused
declare player.number[2] with network priority low -- unused
declare player.speed            with network priority local
declare player.speed_kph        with network priority local
declare player.distance_to_goal with network priority local
declare player.number[6] with network priority local -- unused
declare player.announced_game_start with network priority low
declare player.object[0] with network priority local -- unused
declare player.vehicle   with network priority low
declare player.timer[0] = 3 -- unused
declare object.distance_to_prev with network priority local
declare object.goal_disallowed  with network priority local
declare object.mine_is_armed    with network priority low
declare object.expiration_timer = 16
declare object.detonation_timer = 2

on init: do
end

for each player do -- round card and announce game start
   current_player.announce_start_timer.set_rate(-100%)
   if game.score_to_win != 0 then 
      current_player.set_round_card_title("Drive through flags to earn points.\r\n%n points to win.", game.score_to_win)
   end
   if game.score_to_win == 0 then 
      current_player.set_round_card_title("Drive through flags to earn points.")
   end
   ui_speed.set_text("%n KPH", hud_player.speed_kph)
   if current_player.announced_game_start == 0 and current_player.announce_start_timer.is_zero() then 
      send_incident(rally_game_start, current_player, no_player)
      current_player.announced_game_start = 1
   end
end

on local: do -- track player speed and distance to goal for UI
   for each player do
      current_player.speed     = current_player.biped.get_speed()
      current_player.speed_kph = current_player.speed
      current_player.speed_kph *= 109
      current_player.speed_kph /= 100
      current_player.distance_to_goal = 0
      current_player.distance_to_goal = current_player.biped.get_distance_to(current_goal)
      current_player.distance_to_goal *= 7
      current_player.distance_to_goal /= 23
   end
end

do -- update player movement stats
   movement_stats_update_interval.set_rate(-100%)
   if movement_stats_update_interval.is_zero() then 
      for each player do
         temp_int_02  = 1
         temp_int_03  = current_player.speed
         temp_int_02 *= temp_int_03
         temp_int_02 *= 109
         temp_int_02 /= 100
         current_player.distance_traveled += temp_int_02
         if current_player.speed_kph > current_player.top_speed then 
            current_player.top_speed = current_player.speed_kph
         end
      end
      movement_stats_update_interval.reset()
   end
end

if current_goal == no_object then -- select next goal
   alias created_anti_zone = temp_obj_00
   --
   for each object with label all_anti_zones do
      current_object.delete()
   end
   current_goal = get_random_object("rally_flag", previous_goal)
   if current_goal.goal_disallowed == 1 then
      --
      -- This goal isn't suitable for use for some reason. We'll try to pick 
      -- another one next frame.
      --
      current_goal = no_object
   end
   if not current_goal == no_object then 
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
      -- Discourage spawning near the current goal (won't work properly 
      -- if the goal is a movable object):
      --
      created_anti_zone = no_object
      created_anti_zone = current_goal.place_at_me(respawn_zone_weak_anti, none, never_garbage_collect, 0, 0, 0, none)
      created_anti_zone.team = neutral_team
      --
      is_after_initial = 1
   end
end

for each object with label "rally_flag" do
   alias anyone_scored = temp_int_02
   if current_goal == current_object then 
      temp_obj_00 = current_object
      anyone_scored = 0
      for each player do
         if current_object.shape_contains(current_player.biped) then 
            temp_obj_01 = no_object
            temp_obj_01 = current_player.try_get_vehicle()
            if not temp_obj_01 == no_object then 
               anyone_scored = 1
               current_player.score += 1
            end
         end
      end
      if anyone_scored == 1 then 
         previous_goal = current_goal -- select a new goal next frame
         current_goal  = no_object
         temp_obj_00.set_waypoint_visibility(no_one)
         temp_obj_00.set_shape_visibility(no_one)
      end
   end
end

-- Keep track of which objects would be unsuitable for use as our next goal
for each object with label "rally_flag" do
   if current_goal == no_object then -- if we need to pick a new goal
      current_object.goal_disallowed = 0
      if not previous_goal == no_object then 
         current_object.distance_to_prev = previous_goal.get_distance_to(current_object)
         if current_object.distance_to_prev < 50 then 
            current_object.goal_disallowed = 1
         end
      end
   end
end
for each player do
   if current_goal == no_object then -- if we need to pick a new goal 
      temp_plr_00 = current_player
      for each object with label "rally_flag" do
         if current_object.shape_contains(temp_plr_00.biped) then 
            current_object.goal_disallowed = 1
         end
      end
   end
end

if game.round_timer.is_zero() then -- round timer (but wouldn't this end the round instantly if the time limit is Unlimited?)
   game.end_round()
end

for each player do -- create vehicles for players that need them
   if current_player.vehicle == no_object then 
      temp_obj_00 = current_player.biped
      if not temp_obj_00 == no_object then 
         current_player.vehicle = current_player.biped.place_at_me(mongoose, none, none, 0, 0, 0, none)
         current_player.force_into_vehicle(current_player.vehicle)
         temp_obj_01      = current_player.vehicle
         temp_obj_01.team = current_player.team
      end
   end
end

for each player do -- track the player's owned vehicle
   temp_obj_00 = no_object
   temp_obj_00 = current_player.try_get_vehicle()
   if not temp_obj_00 == no_object then 
      current_player.vehicle = temp_obj_00
   end
end

for each player do -- abandon vehicles when their owners are killed
   if current_player.killer_type_is(guardians | suicide | kill | betrayal | quit) then 
      current_player.vehicle = no_object
   end
end

for each object with label "none" do
   current_object.delete()
end

for each object with label all_landmines do
   alias current_landmine = temp_obj_00
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

for each player do
   ui_speed.set_visibility(current_player, true)
   temp_obj_00 = no_object
   temp_obj_00 = current_player.try_get_vehicle()
   if temp_obj_00 == no_object then 
      ui_speed.set_visibility(current_player, false)
   end
end

for each object with label all_mongooses do -- expire mongooses if they move too slowly for too long
   alias speed = temp_int_02
   --
   speed = 0
   speed = current_object.get_speed()
   if speed < 50 then 
      current_object.expiration_timer.set_rate(-100%)
      if current_object.expiration_timer.is_zero() then 
         current_object.delete()
      end
   end
end
for each object with label all_mongooses do -- ...unless they're near any player
   alias distance = temp_int_02
   --
   temp_obj_00 = no_object -- not used in this block
   distance = 0
   for each player do
      distance = current_object.get_distance_to(current_player.biped)
      if distance < 10 then 
         current_object.expiration_timer.reset()
      end
   end
end
