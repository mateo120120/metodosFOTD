
data = readtable('data_motor.csv'); 
t = data{:,2}; % Tiempo
u = data{:,3}; % Entrada (escalón)
y = data{:,4}; % Respuesta del sistema


% Linea de 100 % y 0%
y_final = mean(y(end-10:end));
y_inicial = 0;

y_63 =  0.63 * y_final;
p_63 = find(y >= y_63, 1);  
t_63 = t(p_63);

y_28 = 0.284 * y_final; 
p_28 = find(y >= y_28 ,1);
t_28 =t(p_28);

% Puntos y ecuacion de la recta

x1 = 0.656566; y1 = 0.334005;
x2 = 0.959596; y2 = 0.61822;
m = (y2 - y1) / (x2 - x1);
b = y1 - m * x1;
xp = linspace(0.309091, 1.34545, 100);
yp = m*xp + b;

% Parametros del modelo FOTD

K = (y_final-y_inicial/max(u)-min(u));
theta = 0.309091;
tau =1.34545- theta;

% 1) Metodo Ziegler Nichols
Gp1 = tf(K, [tau 1],'InputDelay', theta);
[yz, tz] = step(Gp1,t);
% ys = lsim(Gp1, u, t);


% 2) Miller
tau2 = t_63 -theta;
Gp2 = tf(K, [tau2 1],'InputDelay', theta);
[ym, tm] = step(Gp2,t);
% ys = lsim(Gp1, u, t);

% 3) Analitico
tau3 = -3 * (t_28 - t_63) / 2;
theta3 = t_63 - tau3;
Gp3 = tf(K, [tau3 1],'InputDelay', theta3);
[ya, ta] = step(Gp3,t);

% Definir el color gris si no está definido
gris = [0.5 0.5 0.5];  
naranja = [1, 0.5, 0];  

% Graficar la respuesta y la línea base
figure;
plot(t, y, 'r-', 'LineWidth',2); hold on;
plot(t, u, 'b-', 'LineWidth',2);
plot(tz, yz, 'g-', 'LineWidth',2);
plot(tm, ym, 'm-', 'LineWidth',2);
plot(ta, ya, 'Color', naranja, 'LineWidth',2);
yline(y_final, 'k--', 'LineWidth',1);
yline(y_inicial, 'r--', 'LineWidth',1);
plot(xp, yp, '--', 'Color', gris, 'LineWidth', 1);

legend({'Respuesta del proceso','Señal del Escalón','Ziegler and Nichols','Miller','Analítico','Linea 100%','Linea Base'}, ...
    'Location','northwest');

xlim([0 5]);
ylim([-0.1 1.6]);
grid on;
