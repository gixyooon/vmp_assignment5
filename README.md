# vmp_assignment5
# demo video linkage : https://youtu.be/FB82lYxriCI 

  import pygame
  import numpy as np
  import time
  
  FPS = 50   # frames per second
  
  WINDOW_WIDTH = 1000
  WINDOW_HEIGHT = 800
  
  class myPolygon():
      def __init__(self, nvertices = 3, radius=70, color=(100,0,0), vel=[5.,0]):
          self.radius = radius
          self.nvertices = nvertices
          self.vertices = getRegularPolygon(self.nvertices, radius=self.radius)
  
          self.color = color
          self.color_org = color 
  
          self.angle = 0.
          self.angvel = np.random.normal(5., 7) #객체의 각도 변화 속도
          # 평균이 5이고 표준 편차가 7인 정규 분포에서 무작위로 값을 선택하여 초기화
  
          self.position = np.array([0.,0]) #
          # self.position = self.vertices.sum(axis=0) # 2d array
          self.vel = np.array(vel)
          self.tick = 0
  
      def update(self,):
          self.tick += 1
          self.angle += self.angvel
          self.position += self.vel
  
          if self.position[0] >= WINDOW_WIDTH:
              self.vel[0] = -1. * self.vel[0]
  
          if self.position[0] < 0:
              self.vel[0] *= -1.
  
          if self.position[1] >= WINDOW_HEIGHT:
              self.vel[1] *= -1.
  
          if self.position[1] < 0:
              self.vel[1] *= -1
  
          # print(self.tick, self.position)
  
          return
  
      def draw(self, screen):
          R = Rmat(self.angle)
          points = self.vertices @ R.T + self.position
  
          pygame.draw.polygon(screen, self.color, points)
  #
  
  def update_list(alist):
      for a in alist:
          a.update()
  #
  def draw_list(alist, screen):
      for a in alist:
          a.draw(screen)
  
  def Rmat(degree): #degree 매개변수로 받은 각도 -> radian 변환 (행렬 회전 연산을 위해)
      rad = np.deg2rad(degree) 
      c = np.cos(rad)
      s = np.sin(rad)
      R = np.array([ [c, -s, 0],
                     [s,  c, 0], [0,0,1]])
      return R
  
  def Tmat(tx, ty):
      Translation = np.array( [
          [1, 0, tx],
          [0, 1, ty],
          [0, 0, 1]
      ])
      return Translation
  
  def draw(P,H,screen,color=(100,200,200)):
      R = H[:2,:2]
      T = H[:2,2]
      Ptransformed = P @ R.T + T
      pygame.draw.polygon(screen, color=color, points=Ptransformed, width=3)
      return
      
  
  def drawings(screen, position):
      H0 = Tmat(position[0],position[1])
      x=H0[0,2]
      y=H0[1,2]
      
      
      
  def main():
      pygame.init() # initialize the engine
  
      screen = pygame.display.set_mode( (WINDOW_WIDTH, WINDOW_HEIGHT))
      clock = pygame.time.Clock()
      
      w=100
      h=20
      X = np.array([[0,0],[w,0],[w,h],[0,h]])
      G = np.array([[0,0],[w/2,0],[w/2,h],[0,h]])
      position = [WINDOW_WIDTH/2, WINDOW_HEIGHT-100]
      
      jointangle1=0
      jointangle2=-0
      jointangle3=0
      jointangle4=0
      jointangle5=-0
      jointangle6=0
      gripperN=0
      gripperON=90
      
      tick=0
      done = False
      while not done:
          #  input handling
          tick+=1
          
          for event in pygame.event.get():
              if event.type == pygame.QUIT:
                  done = True
                  
          keys = pygame.key.get_pressed()
          
          #arm 1
          if keys[pygame.K_z]:  
              jointangle1 -= 1  
          if keys[pygame.K_x]:  
              jointangle1 += 1  
          
          #arm 2
          if keys[pygame.K_a]:  
              jointangle2 -= 1  
          if keys[pygame.K_s]: 
              jointangle2 += 1 
             
          #arm 3 
          if keys[pygame.K_q]: 
              jointangle3 -= 1 
          if keys[pygame.K_w]: 
              jointangle3 += 1
          
          #arm 4  
          if keys[pygame.K_c]: 
              jointangle4 -= 1 
          if keys[pygame.K_v]: 
              jointangle4 += 1
              
          #arm 5
          if keys[pygame.K_d]: 
              jointangle5 -= 1 
          if keys[pygame.K_f]: 
              jointangle5 += 1
          
          #arm 6
          if keys[pygame.K_e]: 
              jointangle6 -= 1 
          if keys[pygame.K_r]: 
              jointangle6 += 1
              
          #gripper
          if keys[pygame.K_SPACE]:
              gripperN += 1
              time.sleep(0.2)
          if gripperN%2==0 and 55<= gripperON <= 90:
              gripperON +=1   
              if gripperON == 91:
                  gripperON=90
          if gripperN%2!=0 and 0<= gripperON <= 90:
              gripperON -=1
              if gripperON == 54:
                  gripperON=55
          
          # drawing
          screen.fill((180,180,180))
          
          #ground
          pygame.draw.rect(screen, (0,100,100),(0,WINDOW_HEIGHT-70,1000,70))
          
          #base
          #pygame.draw.circle(screen, (255,0,0), position, radius=3)
          H0=Tmat(position[0]-w/2, position[1]) @Tmat(0,h/2)
          draw(X, H0, screen, (0,0,0)) #base
          
          #arm 1
          H1 = H0 @ Tmat(w/2,0)
          x,y = H1[0,2],H1[1,2] #joint position
          H11= H1 @ Rmat(-90) @Rmat(jointangle1) @Tmat(0, -h/2) 
          pygame.draw.circle(screen,(0,240,200),(x,y),radius=h/2) #joint
          draw(X,H11,screen,(50,50,50)) #arm1, 90 degree
          
          #arm 2
          H2 = H11 @ Tmat(w,0) @Tmat(0,h/2)
          x,y = H2[0,2],H2[1,2]
          pygame.draw.circle(screen, (0,240,200), (x,y), radius=h/2)
          H22= H2 @Rmat(jointangle2) @Tmat(0,-h/2)
          draw(X,H22,screen,(50,50,50))
          
          #arm 3
          H3 = H22 @ Tmat(w,0) @Tmat(0,h/2)
          x,y = H3[0,2],H3[1,2]
          pygame.draw.circle(screen, (0,240,200), (x,y), radius=h/2)
          H33= H3 @Rmat(jointangle3) @Tmat(0,-h/2)
          draw(X,H33,screen,(50,50,50))
          
          #arm 4
          H4 = H33 @ Tmat(w,0) @ Tmat(0,h/2)
          x,y = H4[0,2], H4[1,2]
          pygame.draw.circle(screen, (0,240,200), (x,y), radius=h/2)
          H44 = H4 @ Rmat(jointangle4) @ Tmat(0,-h/2)
          draw(X, H44, screen, (50,50,50))
  
          #arm 5
          H5 = H44 @ Tmat(w,0) @ Tmat(0,h/2)
          x,y = H5[0,2], H5[1,2]
          pygame.draw.circle(screen, (0,240,200), (x,y), radius=h/2)
          H55 = H5 @ Rmat(jointangle5) @ Tmat(0,-h/2)
          draw(X, H55, screen,(50,50,50))
  
          #arm 6
          H6 = H55 @ Tmat(w,0) @ Tmat(0,h/2)
          x,y = H6[0,2], H6[1,2]
          pygame.draw.circle(screen, (0,240,200), (x,y), radius=h/2)
          H66 = H6 @ Rmat(jointangle6) @ Tmat(0,-h/2)
          draw(X, H66, screen,(50,50,50))
          
          
          #gripper bodies
          H7 = H66 @ Tmat(w,0) @Tmat(0,h/2)
          x,y = H7[0,2],H7[1,2]
          pygame.draw.circle(screen, (0,240,200), (x,y), radius=h/2)
          H77= H7 @Rmat(gripperON) @Tmat(0,-h/2)
          draw(G,H77,screen,(50,50,50))
          
          H777= H7 @Rmat(-gripperON) @Tmat(0,-h/2)
          draw(G,H777,screen,(50,50,50))
          
          #gripper arms
          H8 = H77 @ Tmat(w/2,0) @Tmat(0,h/2) @Rmat(-90)
          x,y = H8[0,2],H8[1,2]
          pygame.draw.circle(screen, (0,240,200), (x,y), radius=h/2)
          H88= H8  @Tmat(0,-h/2)
          draw(G,H88,screen,(50,50,50))
          
          H888 = H777 @ Tmat(w/2,0) @Tmat(0,h/2) @Rmat(90) #joint3
          x,y = H888[0,2],H888[1,2]
          pygame.draw.circle(screen, (0,240,200), (x,y), radius=h/2)
          H8888= H888  @Tmat(0,-h/2)
          draw(G,H8888,screen,(50,50,50))
          
          
          # finish
          pygame.display.flip()
          clock.tick(FPS)
      # end of while
  # end of main()
  
  if __name__ == "__main__":
      main()
