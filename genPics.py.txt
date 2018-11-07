import bpy
import mathutils
from math import radians
from mathutils import Vector
import time

#Higher numbers mean further away camera
#should be manually tuned
CAM_DIST_MULT = 4.5

'''
number of pictures, note that due to the
way the pictures are created that the alg
will actually generate a number of pictures equal
to closest perfect cube of the number provided
'''
NUM_PICS = 1000
PI = radians(180)

#the location you wish to save the files
SAVE_PATH = "C:/Users/ExampleUser/ExampleFolder"

step_count = round(NUM_PICS ** (1./3))
print (step_count)

cam = bpy.data.objects['Camera']
maxVertDist = 0
sceneCenter = Vector([0,0,0])
numObj=0
scene = bpy.context.scene

cam.rotation_euler = Vector([0,0,0])

def findMax(vertCoords):
	global maxVertDist
	vectCoords = Vector(vertCoords)
	if((sceneCenter - vectCoords).length > maxVertDist):
		maxVertDist = (sceneCenter - vectCoords).length

#point camera towards center of the screen
def pointCam(point):
	scene.update()
	loc_camera = cam.matrix_world.to_translation()
	direction = point - loc_camera
	rot_quat = direction.to_track_quat('-Z', 'Y')
	cam.rotation_euler = rot_quat.to_euler()


#iterate through all the objects in the scene and calculate the center
for obj in bpy.data.objects:
    if obj.type == 'MESH':
        vcos = [ obj.matrix_world * v.co for v in obj.data.vertices ]
        findCenter = lambda l: ( max(l) + min(l) ) / 2

        x,y,z  = [ [ v[i] for v in vcos ] for i in range(3) ]
        center = [ findCenter(axis) for axis in [x,y,z] ]
        sceneCenter = sceneCenter + Vector(center)
        numObj+=1
        
sceneCenter = sceneCenter / numObj


for obj in bpy.data.objects:
	if obj.type == 'MESH':
		vcos = [ obj.matrix_world * v.co for v in obj.data.vertices ]
		[ findMax(vert) for vert in vcos ]
		

camDist = maxVertDist * CAM_DIST_MULT




empty = None

if (bpy.data.objects.get("empty") == None):
	empty = bpy.data.objects.new("empty",None)
	scene.objects.link(empty)
else:
	cam.parent = None
	empty = bpy.data.objects['empty']




empty.location = sceneCenter
empty.rotation_euler = (0,0,0)
cam.parent = empty



scene.cursor_location = sceneCenter
index = 0
cam.location = sceneCenter + Vector([camDist,0,0])

pointCam(sceneCenter)



for stepx in range(0, step_count):
	for stepy in range(0, step_count):
		for stepz in range(0, step_count):
			x_rot = (stepx/step_count)*2*PI
			y_rot = (stepy/step_count)*2*PI
			z_rot = (stepz/step_count)*2*PI
			empty.rotation_euler = (x_rot,y_rot,z_rot)
			bpy.data.scenes["Scene"].render.filepath =  SAVE_PATH + str(index)
			bpy.ops.render.render( write_still=True )
			index+=1

cam.parent = None
            

