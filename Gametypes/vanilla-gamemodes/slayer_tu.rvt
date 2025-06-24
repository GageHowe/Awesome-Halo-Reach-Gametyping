
alias opt_hidden_gametype = script_option[0]
alias hidden_gametype_covy = 3
alias hidden_gametype_swat = 7
alias hidden_gametype_bro  = 8
alias opt_kill_points       = script_option[1]
alias opt_death_points      = script_option[2]
alias opt_suicide_points    = script_option[3]
alias opt_betrayal_points   = script_option[4]
alias opt_leader_kill_bonus = script_option[5]
alias opt_headshot_bonus    = script_option[6]
alias opt_pummel_bonus      = script_option[7]
alias opt_assassin_bonus    = script_option[8]
alias opt_splatter_bonus    = script_option[9]
alias opt_sticky_bonus      = script_option[10]
alias opt_spree_bonus       = script_option[11]
alias opt_super_shields     = script_option[12]

alias leader_traits = script_traits[0]

-- Forge labels:
alias all_health_packs   = 1
alias all_fire_particles = 3
alias all_jetpacks       = 6
alias all_armor_locks    = 7

alias rating_stat = player.script_stat[0]

alias announced_game_start = player.number[1]
alias announce_start_timer = player.timer[0]

alias temp_int_00 = global.number[0]
alias temp_int_01 = global.number[1]
alias temp_int_02 = global.number[2]
alias temp_int_03 = global.number[3]
alias temp_obj_00 = global.object[0]
alias temp_plr_00 = global.player[0]
alias lifespan    = object.timer[0] -- for super shields particles
alias is_leader   = player.number[0]
alias ach_top_shot_count        = player.number[2]
alias ach_license_to_kill_count = player.number[3]
alias last_biped  = player.object[0]
alias ach_paper_beats_rock_vuln_timer = player.timer[1]

declare temp_int_00 with network priority local
declare temp_int_01 with network priority local
declare temp_int_02 with network priority local
declare temp_int_03 with network priority local
declare temp_obj_00 with network priority local
declare temp_plr_00 with network priority local
declare global.player[1] with network priority local -- effectively unused; only exists due to Bungie typos
declare player.is_leader with network priority local
declare player.announced_game_start with network priority low
declare player.ach_top_shot_count        with network priority low
declare player.ach_license_to_kill_count with network priority low
declare player.last_biped with network priority low
declare player.announce_start_timer = 5
declare object.lifespan = 1

if game.teams_enabled == 1 then -- enforce FFA-only label
   for each object with label "ffa_only" do
      current_object.delete()
   end
end

if game.teams_enabled == 0 then -- enforce team-only label
   for each object with label "team_only" do
      current_object.delete()
   end
end

on init: do
end

for each player do -- loadout palettes
   if current_player.is_elite() then 
      current_player.set_loadout_palette(elite_tier_1)
   end
   if not current_player.is_elite() then 
      current_player.set_loadout_palette(spartan_tier_1)
   end
end

for each player do -- round card
   if game.score_to_win != 0 and game.teams_enabled == 1 then 
      current_player.set_round_card_title("Kill players on the enemy team.\r\n%n points to win.", game.score_to_win)
   end
   if game.score_to_win != 0 and game.teams_enabled == 0 then 
      current_player.set_round_card_title("Score points by killing other players.\r\n%n points to win.", game.score_to_win)
   end
   if game.score_to_win == 0 and game.teams_enabled == 1 then 
      current_player.set_round_card_title("Kill players on the enemy team.")
   end
   if game.score_to_win == 0 and game.teams_enabled == 0 then 
      current_player.set_round_card_title("Score points by killing other players.")
   end
end

for each team do -- Buddy Slayer spawning mode setup
   if opt_hidden_gametype == hidden_gametype_bro then 
      current_team.set_co_op_spawning(true)
   end
end
for each object with label "bro_spawn_loc" do -- Buddy Slayer spawn permission setup
   if opt_hidden_gametype == hidden_gametype_bro then 
      current_object.set_invincibility(1)
      current_object.set_pickup_permissions(no_one)
      current_object.set_spawn_location_fireteams(all)
      current_object.set_spawn_location_permissions(allies)
      for each player do
         if current_object.team == current_player.team then 
            temp_int_01 = 0
            current_player.set_primary_respawn_object(current_object)
         end
      end
   end
end

for each player do -- announce game start
   current_player.announce_start_timer.set_rate(-100%)
   if current_player.announced_game_start == 0 and current_player.announce_start_timer.is_zero() then 
      if opt_hidden_gametype == hidden_gametype_swat then 
         send_incident(swat_game_start, current_player, no_player)
      end
      if not opt_hidden_gametype == hidden_gametype_swat then 
         send_incident(game_start_slayer, current_player, no_player)
      end
      current_player.announced_game_start = 1
   end
end

for each player do -- identify player in the lead, and ensure Arena rating appears in scoreboard if appropriate
   temp_int_01 = 0
   current_player.rating_stat = current_player.rating
   current_player.is_leader = 0
   if game.teams_enabled == 1 then 
      temp_int_01 = current_player.team.get_scoreboard_pos()
   end
   if game.teams_enabled == 0 then 
      temp_int_01 = current_player.get_scoreboard_pos()
   end
   if temp_int_01 == 1 and not current_player.score == 0 then 
      current_player.is_leader = 1
   end
end

-- Delete all Health Packs in Covy Slayer and SWAT:
if opt_hidden_gametype == hidden_gametype_covy or opt_hidden_gametype == hidden_gametype_swat then 
   for each object with label all_health_packs do
      current_object.delete()
   end
end

for each player do -- track player's last biped for achievement stuff; award kill points, death points, and all bonus points
   temp_int_01 = 0
   --
   -- If one player kills another player with a DMR, we want to check how far apart 
   -- they were so we can award the Cross-Mappin' achievement if appropriate. However, 
   -- a player's biped property is cleared when they die. The biped itself is still 
   -- accessible to script, but not via the property. Accordingly, we want to remember 
   -- a player's last known biped so we can make use of it when the player dies:
   --
   temp_obj_00 = current_player.biped
   if temp_obj_00 != no_object then 
      current_player.last_biped = temp_obj_00
   end
   --
   -- And now, the kill/death handling. Note that suicides and betrayals are handled 
   -- in a separate trigger for some reason.
   --
   if current_player.killer_type_is(guardians | suicide | kill | betrayal | quit) then 
      current_player.score += opt_death_points
      if current_player.killer_type_is(kill) then 
         alias killer = temp_plr_00
         alias victim = current_player
         --
         killer = victim.try_get_killer()
         killer.score += opt_kill_points
         do -- handle DLC achievement: Cross-Mappin'
            alias distance = temp_int_02
            --
            distance = 0
            temp_int_03 = 0
            temp_int_03 = victim.try_get_death_damage_type()
            if temp_int_03 == 6 then -- DMR
               distance = victim.last_biped.get_distance_to(killer.last_biped)
               if distance > 400 then 
                  send_incident(dlc_achieve_5, killer, victim)
               end
            end
         end
         temp_int_00 = victim.try_get_death_damage_mod()
         temp_int_01 = killer.get_spree_count()
         do
            temp_int_01 %= 5
            if temp_int_01 == 0 then 
               killer.score += opt_spree_bonus
            end
         end
         if victim.is_leader == 1 then 
            killer.score += opt_leader_kill_bonus
         end
         if temp_int_00 == enums.damage_reporting_modifier.pummel then 
            killer.score += opt_pummel_bonus
         end
         if temp_int_00 == enums.damage_reporting_modifier.assassination then 
            killer.score += opt_assassin_bonus
         end
         if temp_int_00 == enums.damage_reporting_modifier.splatter then 
            killer.score += opt_splatter_bonus
         end
         if temp_int_00 == enums.damage_reporting_modifier.sticky then 
            killer.score += opt_sticky_bonus
         end
         if temp_int_00 == enums.damage_reporting_modifier.headshot then 
            killer.score += opt_headshot_bonus
         end
      end
   end
end

for each player do -- award suicide points
   if current_player.killer_type_is(guardians | suicide | kill | betrayal | quit) and not current_player.killer_type_is(kill) and not current_player.killer_type_is(betrayal) then 
      current_player.score += opt_suicide_points
   end
end

for each player do -- award betrayal points
   alias killer = temp_plr_00
   --
   if current_player.killer_type_is(betrayal) then 
      killer = current_player.try_get_killer()
      killer.score += opt_betrayal_points
   end
end

if game.round_time_limit > 0 and game.round_timer.is_zero() then -- round timer
   game.end_round()
end

for each player do -- apply leader traits
   if current_player.is_leader == 1 then 
      current_player.apply_traits(leader_traits)
   end
end

for each player do -- handle super shields VFX
   if opt_super_shields == 1 then 
      alias current_shields = temp_int_01
      --
      current_shields = 0
      current_shields = current_player.biped.shields
      if current_shields > 100 then 
         temp_int_02 = 0
         temp_int_02 = rand(10)
         if temp_int_02 <= 2 then -- 30% chance to spawn a particle emitter
            current_player.biped.place_at_me(particle_emitter_fire, none, never_garbage_collect, 0, 0, 0, none)
         end
      end
   end
end
for each object with label all_fire_particles do -- clean up super shields VFX
   current_object.lifespan.set_rate(-100%)
   if current_object.lifespan.is_zero() then 
      current_object.delete()
   end
end

--
-- Triggers below handle several DLC achievements; however, there's a mistake that 
-- is made very, very frequently. Some trigger functions are capable of just... not 
-- returning a value. Like, there are situations where this
--
--    global.player[0] = current_player.try_get_killer()
--
-- doesn't actually change the value of the variable it's being assigned to. Bungie 
-- and 343i deal with these cases by clearing the variable first
--
--    global.player[0] = no_player
--    global.player[0] = current_player.try_get_killer()
--
-- while my Megalo language offers variant functions which do that for you.
--
--    global.player[0] = current_player.get_killer() -- compiles the "clear" line
--
-- However, almost all of the triggers below clear the wrong variable!
--
--    global.player[1] = no_player -- Bungie, no!
--    global.player[0] = current_player.try_get_killer()
--
-- 343i's Freeze Tag contains mistake-free versions of all of the triggers below.
--

for each player do -- award Dive Bomber achievement as appropriate
   if current_player.killer_type_is(kill) then 
      alias killer    = temp_plr_00
      alias killer_aa = temp_obj_00
      --
      global.player[1] = no_player -- I think they meant to use (killer) here; bad copying and pasting, maybe?
      killer = current_player.try_get_killer()
      temp_int_01 = 0 -- wrong variable again
      temp_int_00 = current_player.try_get_death_damage_mod()
      if temp_int_00 == enums.damage_reporting_modifier.assassination then -- beatdown i.e. punching an enemy in the back
         killer_aa = no_object
         killer_aa = killer.try_get_armor_ability()
         if killer_aa.has_forge_label(all_jetpacks) and killer_aa.is_in_use() then 
            send_incident(dlc_achieve_2, killer, killer, 65)
         end
      end
   end
end

for each player do -- award From Hell's Heart achievement as appropriate
   if current_player.killer_type_is(kill) then 
      temp_int_01 = 0 -- I think they meant to use temp_int_00 here; bad copying and pasting, maybe?
      temp_int_00 = current_player.try_get_death_damage_mod()
      if temp_int_00 == enums.damage_reporting_modifier.sticky then -- sticky grenade
         global.player[1] = no_player -- wrong variable again
         temp_plr_00 = current_player.try_get_killer()
         if temp_plr_00.killer_type_is(suicide) then 
            send_incident(dlc_achieve_2, current_player, current_player, 68)
         end
      end
   end
end

for each player do -- manage and award Top Shot achievement as appropriate
   if current_player.killer_type_is(guardians | suicide | kill | betrayal | quit) then 
      current_player.ach_top_shot_count = 0
      if current_player.killer_type_is(kill) then 
         alias killer = temp_plr_00
         --
         global.player[1] = no_player -- yet ANOTHER mismatched variable
         killer = current_player.try_get_killer()
         temp_int_01 = 0 -- wrong variable again
         temp_int_00 = current_player.try_get_death_damage_mod()
         if temp_int_00 != enums.damage_reporting_modifier.headshot then -- if not headshot
            killer.ach_top_shot_count = 0
         end
         if temp_int_00 == enums.damage_reporting_modifier.headshot then -- if headshot
            killer.ach_top_shot_count += 1
            if killer.ach_top_shot_count > 2 then 
               send_incident(dlc_achieve_2, killer, killer, 62)
            end
         end
      end
   end
end

for each player do -- manage and award License To Kill achievement as appropriate
   if current_player.killer_type_is(kill) then 
      alias killer         = temp_plr_00
      alias killer_vehicle = temp_obj_00
      --
      global.player[1] = no_player -- wrong variable again...
      killer = current_player.try_get_killer()
      temp_int_01 = 0 -- wrong variable again...
      temp_int_00 = current_player.try_get_death_damage_mod()
      killer_vehicle = no_object
      killer_vehicle = killer.try_get_vehicle()
      if killer_vehicle != no_object and temp_int_00 == enums.damage_reporting_modifier.splatter then -- splatter
         killer.ach_license_to_kill_count += 1
         if killer.ach_license_to_kill_count > 4 then 
            --
            -- Nesting this condition under the previous is marginally more efficient 
            -- than what Freeze Tag does.
            --
            send_incident(dlc_achieve_2, killer, killer, 66)
         end
      end
   end
end

for each player do -- manage timing for the Paper Beats Rock achievement
   alias current_ability = temp_obj_00
   --
   current_ability = no_object
   current_ability = current_player.try_get_armor_ability()
   if current_ability.has_forge_label(all_armor_locks) and current_ability.is_in_use() then 
      current_player.ach_paper_beats_rock_vuln_timer = 4
      current_player.ach_paper_beats_rock_vuln_timer.set_rate(-100%)
   end
end
for each player do -- award Paper Beats Rock achievement as appropriate
   if current_player.killer_type_is(kill) and not current_player.ach_paper_beats_rock_vuln_timer.is_zero() then 
      temp_int_01 = 0 -- wrong variable
      temp_int_00 = current_player.try_get_death_damage_mod()
      if temp_int_00 == enums.damage_reporting_modifier.assassination then 
         global.player[1] = no_player -- wrong variable
         temp_plr_00 = current_player.try_get_killer()
         send_incident(dlc_achieve_2, global.player[0], global.player[0], 60)
      end
   end
end
