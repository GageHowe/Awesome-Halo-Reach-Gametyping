
alias opt_hidden_gametype = script_option[0]
alias hidden_gametype_covy = 3
alias hidden_gametype_swat = 7
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

alias leader_traits = script_traits[0]

-- Forge labels:
alias all_health_packs = 1

alias rating_stat = player.script_stat[0] -- used to show Arena ratings in the post-game carnage report

alias announce_start_timer = player.timer[0]
alias announced_game_start = player.number[1]

alias temp_int_00 = global.number[0]
alias temp_int_01 = global.number[1]
alias temp_plr_00 = global.player[0]
alias is_leader   = player.number[0]
alias last_biped  = player.object[0] -- player.biped is cleared upon the biped's death, so we need to remember it manually

declare global.number[0] with network priority local
declare global.number[1] with network priority local
declare global.number[2] with network priority local
declare global.number[3] with network priority local
declare global.object[0] with network priority local
declare temp_plr_00                 with network priority local
declare player.is_leader            with network priority local
declare player.announced_game_start with network priority low
declare player.last_biped           with network priority low
declare player.announce_start_timer = 5

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
   current_player.is_leader   = 0
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
   global.object[0] = current_player.biped
   if global.object[0] != no_object then
      current_player.last_biped = global.object[0]
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
            alias distance = global.number[2]
            --
            distance = 0
            global.number[3] = 0
            global.number[3] = victim.try_get_death_damage_type()
            if global.number[3] == 6 then -- DMR
               distance = victim.last_biped.get_distance_to(killer.last_biped)
               if distance > 400 then -- 40 Forge units
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
         if temp_int_00 == 1 then 
            killer.score += opt_pummel_bonus
         end
         if temp_int_00 == 2 then 
            killer.score += opt_assassin_bonus
         end
         if temp_int_00 == 3 then 
            killer.score += opt_splatter_bonus
         end
         if temp_int_00 == 4 then 
            killer.score += opt_sticky_bonus
         end
         if temp_int_00 == 5 then 
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
