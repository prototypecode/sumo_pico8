# sumo_pico8

```lua

--p.1 = arrow keys
--p.2 = e d s f

actors = {}
particles = {}
game_m = 0
t = 0
t_off = 0
t_fade = 4
starting = false
is_alive = {0,1}
respawn_timer = {0,1}
is_alive[0] = true
is_alive[1] = true
respawn_timer[0] = 0
respawn_timer[1] = 0
p1_score = 0
p2_score = 0
circ_col = 6
win_time = 90
text_col = 9

function draw_title()
	cls()
	
	local i = 0
	
	while i < t do
		spr(190 + i, 15 * i, 54 + t_off * i)
		i += 1
	end
	
	print("2 players", 37, 64,text_col)
	print("press x", 42, 72)
	print("demo", 48, 120)
		
end

function title_screen()
	t += 1
	if(btn(5) and not starting) then
		sfx(0, 1)
		starting = true
	end
	if (starting) then
		text_col = 0
		t_off += -2
		if (t_off < -80)then
			game_m = 1
		end
	else
	text_col = 7
	end
end

function win_screen()
	if (win_time > 0) then
		win_time -= 1
	
	else
		run()
	end
end

function draw_score()
	print(p1_score, 0, 0)
	print(p2_score, 120, 0)
end

function draw_win()
cls()
if (game_m == 2) then
		print("blue wins", 12, 64)
	end
	if (game_m == 3) then
		print("red wins", 12, 64)
	end
end
function create_actor(x, y, img, n)
	local p = {}
	p.x = x
	p.y = y
	p.dx = 0
	p.dy = 0
	p.img = img
	p.n = n
	p.h = 3
	p.w = 3
	add(actors, p)
	return p
end

function create_part(x, y, img)
	local part = {}
	part.x = x
	part.y = y
	part.img = img
	local posnegx = flr(rnd(3))
	 if (posnegx == 0) then
		 posnegx = -1
	 else
		 posnegx = 1
	 end
	local posnegy = flr(rnd(3))
	 if (posnegy == 0) then
		 posnegy = -1
	 else
		 posnegy = 1
	 end
	part.dx = (2+rnd(4)) * posnegx
	part.dy = (2+rnd(4)) * posnegy
	add(particles,part)
	return part
	
end

function draw_actor(p)
	spr(p.img, p.x, p.y)
end

function draw_part(p)
	spr(p.img, p.x, p.y)
end

function move_actor(p)
	
	p.dx += (0 - p.dx) * 0.05
	p.dy += (0 - p.dy) * 0.05
	
	if(btn(0, p.n)) then
		p.dx += (-4 - p.dx) * 0.1
	end
	if(btn(1, p.n)) then
		p.dx += (4 - p.dx) * 0.1
	end
	if(btn(2, p.n)) then
		p.dy += (-4 - p.dy) * 0.1
	end
	if(btn(3, p.n)) then
		p.dy += (4 - p.dy) * 0.1
	end
	--if not solid_actor(p,p.hsp,0)
	--then
	collide_actor(p, p.dx, 0)
	collide_actor(p, 0, p.dy)
	p.x += p.dx
	
	--end
 
	
	p.y += p.dy
	
end

function move_part(p)
	p.x += p.dx
	p.y += p.dy
	if (p.x > 127 or p.x < 0
	or p.y > 127 or p.y < 0) then
		del(particles, p)
	end
	
end

function destroy_player(p)
	is_alive[p.n] = false
	respawn_timer[p.n] = 60
	if (p.n == 0) then
		p2_score += 1
		circ_col = 14
	else
		p1_score += 1
		circ_col = 12
	end
	sfx(9, 3)
	for i = 0, 8 do
		part = create_part(p.x, p.y, 4)
	end
	del(actors, p)
	
end

function respawn_clock(p)
	if not(is_alive[p]) then
		if(respawn_timer[p] > 0) then
			respawn_timer[p] -= 1
		else
			if(p == 0) then
				p1 = create_actor(32, 64, 2, 0)
				is_alive[0] = true
				circ_col = 6
			else
				p2=create_actor(96, 64, 3, 1)
				is_alive[1] = true
				circ_col = 6
			end
		end
  end
end

function dist_from_center(p)
local d = 0
	 if not(p.x == 64 and p.y == 64) then
	 	d = sqrt( (64-p.x)^2 + (64-p.y)^2 )
	 	else
	 	d = 0
	 end
	 if (d > 54) then
			destroy_player(p)
		end
end


function collide_actor(a, dx, dy)
 for a2 in all(actors) do
 	if a2 != a then
 		local x=(a.x+dx)-a2.x
 		local y=(a.y+dy)-a2.y
 		if((abs(x)<(a.w+a2.w))and
 		(abs(y)<(a.h+a2.h))) then
 			if(dx != 0 and abs(x) <
 			abs(a.x-a2.x)) then
 				v=a.dx+a2.dy
 				a.dx=-v*2
 				a2.dx=v*2
 				sfx(flr(rnd(4)),3)
 				return true
 			end
 			
 			if(dy != 0 and abs(y) <
 				abs(a.y-a2.y)) then
 			v = a.dy+a2.dx
 			a.dy=-v*2
 			a2.dy=v*2
 			sfx(flr(rnd(4)),3)
 			return true
 			end
 		end
 	end
 end
 return false

end

function check_score()
	if p1_score > 9 then
		game_m = 2
		sfx(11,3)
	end
	if p2_score > 9 then
		game_m = 3
		sfx(11,3)
	end
end

function _init()
	p1 = create_actor(32,64,2,0)
 p2 = create_actor(96,64,3,1)
end

function _update()
	if(game_m == 1) then
	foreach(actors,move_actor)
	foreach(actors,dist_from_center)
	foreach(particles,move_part)
	check_score()
	 for i=0,1 do
			respawn_clock(i)
		end
	end
	if(game_m == 0) then
	title_screen()
	end
	if(game_m == 2 or game_m == 3) then
	win_screen()
	end
end

function _draw()
 if (game_m == 1) then
 	cls()
 	circfill(67, 67, 48, circ_col)
		foreach(actors, draw_actor)
		foreach(particles, draw_part)
		draw_score()
	end
	
	if (game_m == 0) then
		draw_title()
	end
	
	if (game_m == 2 or game_m == 3) then
		draw_win()
	end
end


```
