clear all; close all;
% FREQUÊNCIA DE AMOSTRAGEM
Fa=11025;
% PERIODO DE AMOSTRAGEM
n = 1:3*Fa;
% PERÍODO FREQÊNCIA DE AMOSTRAGEM
T=1/Fa;
% FREQUÊNCIA FUNDAMENTAL
f0 = 175 + 75*sin(2*pi*n*T*1/3);

% Frequências de Formantes e Bandas para vogais
Formantes = {[617.193994  1158.212425  2206.205879  3531.819209], ...
             [496.08  1896.12  2161.53  3734.36], ...
             [347.05 1953.07 2320.75 3842.94 ], ...
             [567.16 889.32 2269.36 3370.87], ...
             [386.63 801.66 2458.37 3745.85]}; 
Bandas = {[169.3727 474.4514 177.6951 669.6347], ...
          [70.0487 322.2303 791.6596 230.0562], ...
          [31.6156 1137.2994 135.0879 163.0158], ...
          [97.1613 93.6816 96.3314 153.9292], ...
          [16.0165 174.5438 120.6067 204.3487]};  
      
% Inicializa vetor para armazenar todos os sinais concatenados
sinal_total = [];

for v = 1:5
    Form = Formantes{v};
    Band = Bandas{v};
    
    % VETOR SINAL COM COMPRIMENTO IGUAL AO SOMATÓRIO DOS PERIODOS GLOTAIS T0
    T0 = 1./f0;
    imp = zeros(size(n));
    i = 1;
    N = length(n);
    
    % TREM DE IMPULSOS
    while i < N
        imp(i) = 1;
        i = i + round(T0(i).*Fa);
    end
    
    % MODELO DO IMPULSO GLOTAL
    a = 0.85;
    B = [0, -a*exp(1)*log(a)];
    A = [1, -2*a, a^2];
    Imp = filter(B, A, imp);
    
    % MODELO DO TRATO VOCAL
    sinal = Imp;
    for j = 1:4
        form = Form(j);
        band = Band(j);
        NUM = 1 - 2*abs(exp(-band*T/2))*cos(2*pi*form*T) + abs(exp(-band*T));
        DEN = [1 -2*abs(exp(-band*T/2))*cos(2*pi*form*T) abs(exp(-band*T))];
        sinal = filter(NUM, DEN, sinal);
    end
    sinal = sinal / max(abs(sinal));
    
    % MODELO DE RADIAÇÃO
    AR = [1];
    BR = [1 -1];
    sinalf = filter(BR, AR, sinal);
    
    % Adiciona o sinal ao vetor total
    sinal_total = [sinal_total; sinalf(:)];
end

% PLOT FREQUêNCIA FUNDAMENTAL
figure;
plot(n/Fa,f0);
xlabel('Tempo (s)');
ylabel('Frequência Fundamental F_0 (Hz)');
title('Variação da Frequência Fundamental F_0 ao Longo do Tempo');
grid on;

% PLOT TREM IMPULSOS
figure
plot(n/Fa, imp)
title("Trem de Impulsos")
xlabel("Tempo (s)")
ylabel("Amplitude")
xlim([0 0.035])  
ylim([0 1.1]) 


% PLOT GERADOR IMPULSOS GLOTAIS
figure
plot(n/Fa, Imp)
title("Gerador de Impulsos Glotais")
xlabel("Tempo (s)")
ylabel("Amplitude")
xlim([0 0.035])  
ylim([0 1.1])

% REPRODUZIR SOM FINAL
sound(sinal_total, Fa);
pause(length(sinal_total)/Fa);

%ESPECTOGRAMA
figure;
spectrogram(sinal_total, 1024, 512, 1024, Fa, 'yaxis');
title('Espectrograma do Sinal Concatenado');
xticks([1.5 4.5 7.5 10.5 13.5]); 
xticklabels({'a', 'e', 'i', 'o', 'u'}); 
xlabel('Vogais');
ylabel('Frequência (kHz)');
grid on;
hold on;
for t = 3:3:12
    xline(t, '--k', 'LineWidth', 1.5); 
end
hold off; 

% PLOT DO SINAL DEPOIS DO MODELO DO TRATO VOCAL
figure;
plot(n/Fa, sinal);
xlabel('Tempo (s)');
ylabel('Amplitude');
title('Sinal Depois do Modelo do Trato Vocal');
grid on;
xlim([0 0.035]);  

% PLOT DO SINAL FINAL CONCATENADO
figure;
plot((1:length(sinal_total))/Fa, sinal_total);
xlabel('Tempo (s)');
ylabel('Amplitude');
title('Sinal Final Concatenado');
grid on;

% PLOT DO SINAL FINAL CONCATENADO
figure;
plot((1:length(sinal_total))/Fa, sinal_total);
xlabel('Tempo (s)');
ylabel('Amplitude');
title('Sinal Final Concatenado');
grid on;
xlim([0 0.035]);

audiowrite('sinal_voz.wav', sinal_total, Fa);
