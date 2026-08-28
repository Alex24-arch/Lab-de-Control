function proyecto3()
% PROYECTO3  Diseno interactivo de compensadores sobre el lugar de las raices.
%

% -----------------------------------------------------------------------

    % --- Estado de la planta ---
    S = struct('z', [], 'p', [], 'Kg', 1, 'K', 1);

    % --- Estado del compensador ---
    %   tipo : 'ninguno' | 'ganancia' | 'pd' | 'adelanto' | 'atraso'
    %   zc   : ceros del compensador
    %   pc   : polos del compensador
    %   Kc   : ganancia del compensador
    %   sd   : ubicacion deseada de los polos dominantes
    C = compVacio();

    EJEMPLOS = { ...
      'A - Doble integrador amortiguado', ...
      'G = 4/[s(s+2)]. Ejemplo clasico de diseno de adelanto', ...
      [], [0, -2], 4 ; ...
      ...
      'B - Tres polos reales', ...
      'G = 1/[(s+1)(s+2)(s+3)]. Lento; se acelera con adelanto', ...
      [], [-1, -2, -3], 1 ; ...
      ...
      'C - Con integrador y polo lejano', ...
      'G = 1/[s(s+1)(s+5)]. Margen de estabilidad estrecho', ...
      [], [0, -1, -5], 1 ; ...
      ...
      'D - Planta de segundo orden lenta', ...
      'G = 1/[(s+0.5)(s+3)]. Sin integrador: error estacionario notable', ...
      [], [-0.5, -3], 1 ; ...
      ...
      'E - Planta inestable', ...
      'G = 1/[s(s-1)]. Polo en el SPD: requiere compensacion para estabilizar', ...
      [], [0, 1], 1 };

    banner();

    while true
        mostrarMenu(S, C);
        op = strtrim(leerTexto('Seleccione una opcion: '));

        switch op
            case '1'
                S = ingresarCeros(S);  C = compVacio();
            case '2'
                S = ingresarPolos(S);  C = compVacio();
            case '3'
                S = cargarEjemplo(S, EJEMPLOS);  C = compVacio();
            case '4'
                if hayPlanta(S), mostrarRacional(S, C); end
            case '5'
                if hayPlanta(S), rootLocus(S, C); end
            case '6'
                if hayPlanta(S)
                    S = editarInteractivo(S, C);
                    C = compVacio();   % cambio la planta: el compensador ya no vale
                end
            case '7'
                if hayPlanta(S), C = disenarCompensador(S, C); end
            case '8'
                if hayPlanta(S) && ~strcmp(C.tipo, 'ninguno')
                    sistemaCompensado(S, C);
                elseif hayPlanta(S)
                    fprintf('\n  >> Primero debe disenar un compensador (opcion 7).\n');
                end
            case '9'
                if hayPlanta(S) && ~strcmp(C.tipo, 'ninguno')
                    compararRespuestas(S, C);
                elseif hayPlanta(S)
                    fprintf('\n  >> Primero debe disenar un compensador (opcion 7).\n');
                end
            case {'0', 'q', 'Q'}
                fprintf('\nFin del programa.\n\n');
                return;
            otherwise
                fprintf('\n  >> Opcion no valida.\n');
        end
    end
end

function C = compVacio()
    C = struct('tipo', 'ninguno', 'zc', [], 'pc', [], 'Kc', 1, 'sd', []);
end

% ===================================================================
% INTERFAZ
% ===================================================================
function banner()
    fprintf('\n');
    fprintf('===================================================================\n');
    fprintf('  EL-5409 Laboratorio de Control Automatico - Proyecto corto 3\n');
    fprintf('  Diseno interactivo de compensadores sobre el lugar de las raices\n');
    fprintf('===================================================================\n');
end

function mostrarMenu(S, C)
    fprintf('\n-------------------------------------------------------------------\n');
    fprintf('  MENU PRINCIPAL\n');
    fprintf('-------------------------------------------------------------------\n');
    fprintf('   1) Ingresar ceros de la planta\n');
    fprintf('   2) Ingresar polos de la planta\n');
    fprintf('   3) Cargar una planta de ejemplo\n');
    fprintf('   4) Ver G(s) en formato racional y ecuacion caracteristica\n');
    fprintf('   5) Dibujar el lugar de las raices\n');
    fprintf('   6) EDITAR polos/ceros con el raton (agregar, arrastrar, borrar)\n');
    fprintf('   7) DISENAR compensador (click en la ubicacion deseada)\n');
    fprintf('   8) Ver sistema compensado y nueva ecuacion caracteristica\n');
    fprintf('   9) Comparar respuestas al escalon (planta vs compensada)\n');
    fprintf('   0) Salir\n');
    fprintf('-------------------------------------------------------------------\n');
    if isempty(S.p)
        fprintf('   [sin planta definida]\n');
    else
        fprintf('   Planta:  polos %s   ceros %s   Kg=%g\n', ...
                vec2str(S.p), vec2str(S.z), S.Kg);
    end
    if ~strcmp(C.tipo, 'ninguno')
        fprintf('   Compensador (%s): %s\n', C.tipo, compStr(C));
    end
end

function ok = hayPlanta(S)
    ok = ~isempty(S.p);
    if ~ok
        fprintf('\n  >> Primero defina los polos de la planta (opcion 2 o 3).\n');
    end
end

% ===================================================================
% ENTRADA DE DATOS
% ===================================================================
function S = ingresarCeros(S)
    fprintf('\n--- Ceros de la planta ---\n');
    fprintf('  Separados por comas. Complejos con i, en pares conjugados.\n');
    fprintf('  ENTER si la planta no tiene ceros.\n');
    S.z = pedirVector('  Ceros: ', true);
end

function S = ingresarPolos(S)
    fprintf('\n--- Polos de la planta ---\n');
    fprintf('  Separados por comas. Complejos con i, en pares conjugados.\n');
    while true
        v = pedirVector('  Polos: ', false);
        if isempty(v)
            fprintf('    >> Debe ingresar al menos un polo.\n');
            continue;
        end
        S.p = v;
        S.Kg = pedirEscalar(sprintf('  Ganancia Kg de la planta [%g]: ', S.Kg), S.Kg, false);
        return;
    end
end

function v = pedirVector(msg, permitirVacio)
    while true
        s = strtrim(leerTexto(msg));
        if isempty(s)
            if permitirVacio, v = []; return;
            else, fprintf('    >> Entrada vacia no permitida.\n'); continue;
            end
        end
        s = strrep(strrep(s, '[', ''), ']', '');
        v = str2num(['[' s ']']);   %#ok<ST2NM>
        if isempty(v) || ~isnumeric(v)
            fprintf('    >> No se pudo interpretar como lista de numeros.\n'); continue;
        end
        if any(~isfinite(v))
            fprintf('    >> Hay valores infinitos o NaN.\n'); continue;
        end
        v = v(:).';
        return;
    end
end

function x = pedirEscalar(msg, actual, permitirCero)
    while true
        s = strtrim(leerTexto(msg));
        if isempty(s), x = actual; return; end
        x = str2double(s);
        if isnan(x) || ~isreal(x) || ~isfinite(x)
            fprintf('    >> Debe ser un numero real finito.\n'); continue;
        end
        if ~permitirCero && x == 0
            fprintf('    >> No puede ser cero.\n'); continue;
        end
        return;
    end
end

function S = cargarEjemplo(S, EJEMPLOS)
    n = size(EJEMPLOS,1);
    fprintf('\n--- Plantas de ejemplo ---\n');
    for k = 1:n
        fprintf('   %d) %s\n      %s\n', k, EJEMPLOS{k,1}, EJEMPLOS{k,2});
    end
    fprintf('   0) Cancelar\n');
    k = str2double(strtrim(leerTexto('Opcion: ')));
    if isnan(k) || k < 1 || k > n || k ~= fix(k)
        fprintf('  >> Cancelado.\n'); return;
    end
    S.z = EJEMPLOS{k,3};  S.p = EJEMPLOS{k,4};  S.Kg = EJEMPLOS{k,5};  S.K = 1;
    fprintf('  >> Planta "%s" cargada.\n', EJEMPLOS{k,1});
end

function s = leerTexto(msg)
    s = input(msg, 's');
    if ~ischar(s), s = ''; end
end

% ===================================================================
% POLINOMIOS Y FORMATO RACIONAL
% ===================================================================
function [num, den, ok, msg] = polinomios(S)
% G(s) = Kg * prod(s-z) / prod(s-p), como coeficientes polinomiales.
    ok = true; msg = '';
    num = S.Kg * poly(S.z);
    den = poly(S.p);
    tn = 1e-9*max(1, max(abs(num)));  td = 1e-9*max(1, max(abs(den)));
    if max(abs(imag(num))) > tn || max(abs(imag(den))) > td
        ok = false;
        msg = ['Los coeficientes resultan complejos. Las raices complejas deben ' ...
               'ingresarse en pares conjugados (ej: -1+2i y -1-2i).'];
        return;
    end
    num = real(num);  den = real(den);
end

function [numOL, denOL] = lazoAbierto(S, C)
% Polinomios de la cadena directa completa: Gc(s)*G(s).
% Si no hay compensador, Gc = 1.
    [num, den] = polinomios(S);
    if strcmp(C.tipo, 'ninguno')
        numOL = num;  denOL = den;
        return;
    end
    numC = C.Kc * poly(C.zc);
    denC = poly(C.pc);
    numOL = real(conv(num, numC));
    denOL = real(conv(den, denC));
end

function c = ecCaracteristica(numOL, denOL, K)
% den(s) + K*num(s) = 0, alineando por la derecha.
    numPad = [zeros(1, numel(denOL)-numel(numOL)), numOL];
    c = denOL + K*numPad;
    idx = find(abs(c) > 1e-12, 1, 'first');
    if isempty(idx), c = 0; return; end
    c = c(idx:end);
end

function mostrarRacional(S, C)
    [num, den, ok, msg] = polinomios(S);
    if ~ok, fprintf('\n  >> %s\n', msg); return; end

    fprintf('\n===================================================================\n');
    fprintf('  FUNCION DE TRANSFERENCIA DE LA PLANTA\n');
    fprintf('===================================================================\n');

    % Forma factorizada (la que ingreso el usuario)
    fprintf('  Forma factorizada:\n');
    fprintf('    G(s) = %g * %s / %s\n\n', S.Kg, factStr(S.z), factStr(S.p));

    % Forma racional (polinomios expandidos)
    fprintf('  Forma racional:\n');
    sNum = poli2str(num);  sDen = poli2str(den);
    anchoLinea = max(numel(sNum), numel(sDen));
    fprintf('           %s\n', sNum);
    fprintf('    G(s) = %s\n', repmat('-', 1, anchoLinea));
    fprintf('           %s\n\n', sDen);

    fprintf('    n = %d polos,  m = %d ceros,  exceso polo-cero = %d\n', ...
            numel(S.p), numel(S.z), numel(S.p)-numel(S.z));

    % Ecuacion caracteristica
    [numOL, denOL] = lazoAbierto(S, C);
    c = ecCaracteristica(numOL, denOL, S.K);
    fprintf('\n  ECUACION CARACTERISTICA:  1 + K*Gol(s) = 0\n');
    if strcmp(C.tipo,'ninguno')
        fprintf('    Gol(s) = G(s)   (sin compensador)\n');
    else
        fprintf('    Gol(s) = Gc(s)*G(s)   con Gc = %s\n', compStr(C));
    end
    fprintf('    den(s) + K*num(s) = 0,  con K = %g:\n', S.K);
    fprintf('      %s = 0\n', poli2str(c));
    fprintf('    Grado %d  ->  %d polos de lazo cerrado\n', numel(c)-1, numel(c)-1);
    fprintf('    Raices actuales: %s\n', vec2str(roots(c).'));
    fprintf('===================================================================\n');
end

% ===================================================================
% LUGAR DE LAS RAICES
% ===================================================================
function [fig, lims] = rootLocus(S, C, silencioso)
% Dibuja el lugar de las raices de 1 + K*Gc(s)*G(s) = 0.
% La ventana se fija a partir de la geometria de polos y ceros, y el
% barrido de K se detiene cuando las ramas salen de ella; si se hiciera al
% reves, con K grande las ramas se irian lejos, la vista se ajustaria a
% ellas y todos los sistemas se verian iguales.
    if nargin < 3, silencioso = false; end

    [numOL, denOL] = lazoAbierto(S, C);
    pOL = roots(denOL);
    zOL = roots(numOL);
    n = numel(pOL);  m = numel(zOL);

    % --- Ventana de interes ---
    pz = [pOL(:); zOL(:)];
    xs = real(pz);  ys = imag(pz);
    L = max([max(xs)-min(xs), 2*max(abs(ys)), 1]);
    xLim = [min(xs)-0.8*L, max(xs)+0.8*L];
    yLim = [-(max(abs(ys))+0.8*L), (max(abs(ys))+0.8*L)];
    Rvista = max(abs([xLim, yLim]));

    % --- K maximo: hasta que las ramas salen de la ventana ---
    Kmax = 1e-3;
    for it = 1:200
        r = roots(ecCaracteristica(numOL, denOL, Kmax));
        if isempty(r) || max(abs(r)) > 1.6*Rvista, break; end
        Kmax = Kmax*1.5;
    end

    Ks = unique([0, linspace(0, Kmax, 1200), Kmax*logspace(-5, 0, 1200)]);
    R = nan(numel(Ks), n);
    for i = 1:numel(Ks)
        r = roots(ecCaracteristica(numOL, denOL, Ks(i)));
        if numel(r) < n, r = [r; nan(n-numel(r),1)]; end %#ok<AGROW>
        R(i,:) = r(1:n).';
    end

    fig = figure('Name', 'Lugar de las raices', 'NumberTitle', 'off');
    hold on; grid on;
    plot(xLim, [0 0], 'k-', 'LineWidth', 0.8);
    plot([0 0], yLim, 'k-', 'LineWidth', 0.8);

    h = []; et = {};
    hR = plot(real(R), imag(R), '.', 'MarkerSize', 4, 'Color', [0 0.45 0.85]);
    h(end+1) = hR(1);  et{end+1} = 'Ramas (K creciente)';

    if n > m
        sigma = (sum(real(pOL)) - sum(real(zOL)))/(n-m);
        primera = true;
        for k = 0:(n-m-1)
            angk = (2*k+1)*pi/(n-m);
            hh = plot([sigma, sigma+3*Rvista*cos(angk)], [0, 3*Rvista*sin(angk)], ...
                      '--', 'Color', [0.6 0.6 0.6]);
            if primera
                h(end+1) = hh; et{end+1} = sprintf('Asintotas (\\sigma_a=%.3g)', sigma);
                primera = false;
            end
        end
    end

    h(end+1) = plot(real(S.p), imag(S.p), 'x', 'MarkerSize', 12, ...
                    'LineWidth', 2.5, 'Color', [0.85 0.10 0.10]);
    et{end+1} = 'Polos de la planta';

    if ~isempty(S.z)
        h(end+1) = plot(real(S.z), imag(S.z), 'o', 'MarkerSize', 10, ...
                        'LineWidth', 2.5, 'Color', [0.10 0.60 0.10]);
        et{end+1} = 'Ceros de la planta';
    end

    % Polos y ceros aportados por el compensador, en otro color
    if ~strcmp(C.tipo,'ninguno')
        if ~isempty(C.pc)
            h(end+1) = plot(real(C.pc), imag(C.pc), 'x', 'MarkerSize', 12, ...
                            'LineWidth', 2.5, 'Color', [0.6 0.2 0.8]);
            et{end+1} = 'Polo(s) del compensador';
        end
        if ~isempty(C.zc)
            h(end+1) = plot(real(C.zc), imag(C.zc), 'o', 'MarkerSize', 10, ...
                            'LineWidth', 2.5, 'Color', [0.9 0.5 0.0]);
            et{end+1} = 'Cero(s) del compensador';
        end
    end

    % Ubicacion deseada y polos de lazo cerrado resultantes
    if ~isempty(C.sd)
        h(end+1) = plot(real([C.sd conj(C.sd)]), imag([C.sd conj(C.sd)]), 'p', ...
                        'MarkerSize', 15, 'MarkerFaceColor', [1 0.85 0], ...
                        'MarkerEdgeColor', 'k');
        et{end+1} = 'Ubicacion deseada s_d';
        yLim = [min(yLim(1), -1.4*abs(imag(C.sd))), max(yLim(2), 1.4*abs(imag(C.sd)))];
        xLim = [min(xLim(1), real(C.sd)-0.5*L), xLim(2)];
    end

    xlim(xLim); ylim(yLim);
    axis equal;                % angulos de asintotas correctos
    xlim(xLim); ylim(yLim);
    xlabel('Eje real  \sigma');  ylabel('Eje imaginario  j\omega');
    if strcmp(C.tipo,'ninguno')
        title(sprintf('Lugar de las raices de la planta   (K: 0 a %.4g)', Kmax));
    else
        title(sprintf('Lugar de las raices COMPENSADO   (K: 0 a %.4g)', Kmax));
    end
    legend(h, et, 'Location', 'best');
    hold off; drawnow;

    lims = [xLim, yLim];
    if ~silencioso
        fprintf('\n  Lugar de las raices generado. Barrido de K: 0 a %.6g\n', Kmax);
    end
end

% ===================================================================
% EDICION INTERACTIVA CON EL RATON
% ===================================================================
function S = editarInteractivo(S, C)
% Permite agregar, arrastrar y borrar polos y ceros haciendo click sobre
% el plano s. Usa ginput(), que existe tanto en MATLAB como en Octave.
%
% Las raices complejas se manejan SIEMPRE en pares conjugados: si se hace
% click fuera del eje real, se agrega automaticamente el conjugado, porque
% de lo contrario los coeficientes del polinomio dejarian de ser reales y
% el sistema no seria fisicamente realizable.

    fprintf('\n--- Edicion interactiva ---\n');
    fprintf('   1) Agregar polo\n');
    fprintf('   2) Agregar cero\n');
    fprintf('   3) Arrastrar (mover) un polo o cero existente\n');
    fprintf('   4) Borrar un polo o cero\n');
    fprintf('   0) Volver\n');
    modo = strtrim(leerTexto('Modo: '));
    if any(strcmp(modo, {'0',''})), return; end

    rootLocus(S, C, true);
    hold on;

    tolSnap = [];   % se calcula segun la escala de la vista
    xl = xlim; yl = ylim;
    tolSnap = 0.03*max(diff(xl), diff(yl));   % umbral para pegar al eje real
    tolSel  = 0.08*max(diff(xl), diff(yl));   % radio de seleccion

    fprintf('\n  Haga click en el grafico. ENTER o click derecho para terminar.\n');
    if any(strcmp(modo, {'3'}))
        fprintf('  Para mover: primer click selecciona, segundo click reubica.\n');
    end

    while true
        [x, y, btn] = capturarClick();
        if isempty(x) || btn == 3, break; end

        % Si el click cae muy cerca del eje real, se interpreta como real:
        % es lo que casi siempre se quiere y evita pares conjugados
        % espurios por imprecision del raton.
        if abs(y) < tolSnap, y = 0; end
        s = x + 1i*y;

        switch modo
            case '1'
                S.p = agregarConConjugado(S.p, s);
                fprintf('   + polo en %s\n', num2strC(s));
            case '2'
                S.z = agregarConConjugado(S.z, s);
                fprintf('   + cero en %s\n', num2strC(s));
            case '3'
                [tipo, idx] = seleccionar(S, s, tolSel);
                if isempty(tipo)
                    fprintf('   (no se selecciono nada cerca del click)\n');
                    continue;
                end
                fprintf('   seleccionado %s en %s -> click en la nueva ubicacion\n', ...
                        tipo, num2strC(getElem(S, tipo, idx)));
                [x2, y2, b2] = capturarClick();
                if isempty(x2) || b2 == 3, break; end
                if abs(y2) < tolSnap, y2 = 0; end
                S = moverElem(S, tipo, idx, x2 + 1i*y2);
                fprintf('   -> movido a %s\n', num2strC(x2+1i*y2));
            case '4'
                [tipo, idx] = seleccionar(S, s, tolSel);
                if isempty(tipo)
                    fprintf('   (no se selecciono nada cerca del click)\n');
                    continue;
                end
                if strcmp(tipo,'polo') && numel(S.p) <= 1
                    fprintf('   >> No se puede borrar: la planta quedaria sin polos.\n');
                    continue;
                end
                val = getElem(S, tipo, idx);
                S = borrarElem(S, tipo, idx);
                fprintf('   - %s en %s eliminado\n', tipo, num2strC(val));
        end

        % Se redibuja para ver el efecto inmediato del cambio.
        close(gcf);
        rootLocus(S, compVacio(), true);
        hold on;
    end

    hold off;
    fprintf('\n  Planta actualizada: polos %s  ceros %s\n', vec2str(S.p), vec2str(S.z));
end

function [x, y, btn] = capturarClick()
% Envuelve ginput para que un entorno sin soporte de raton no rompa el
% programa: en ese caso se piden las coordenadas por teclado.
    x = []; y = []; btn = 1;
    try
        [x, y, btn] = ginput(1);
        if isempty(x), return; end
        if isempty(btn), btn = 1; end
    catch
        fprintf('   (ginput no disponible: ingrese las coordenadas por teclado)\n');
        s = strtrim(leerTexto('   sigma, omega (ENTER para terminar): '));
        if isempty(s), x = []; return; end
        v = str2num(['[' s ']']); %#ok<ST2NM>
        if numel(v) < 2, x = []; return; end
        x = v(1); y = v(2); btn = 1;
    end
end

function v = agregarConConjugado(v, s)
% Agrega s al vector; si s es complejo agrega tambien su conjugado.
    if abs(imag(s)) < 1e-12
        v = [v, real(s)];
    else
        v = [v, s, conj(s)];
    end
end

function [tipo, idx] = seleccionar(S, s, tol)
% Encuentra el polo o cero mas cercano al punto s dentro de un radio tol.
    tipo = ''; idx = [];
    mejor = tol;
    for k = 1:numel(S.p)
        d = abs(S.p(k) - s);
        if d < mejor, mejor = d; tipo = 'polo'; idx = k; end
    end
    for k = 1:numel(S.z)
        d = abs(S.z(k) - s);
        if d < mejor, mejor = d; tipo = 'cero'; idx = k; end
    end
end

function val = getElem(S, tipo, idx)
    if strcmp(tipo,'polo'), val = S.p(idx); else, val = S.z(idx); end
end

function S = moverElem(S, tipo, idx, nuevo)
% Mueve un polo o cero. Si el elemento formaba parte de un par conjugado,
% se mueve el par completo para conservar coeficientes reales.
    val = getElem(S, tipo, idx);
    S = borrarElem(S, tipo, idx);
    if strcmp(tipo,'polo')
        S.p = agregarConConjugado(S.p, nuevo);
    else
        S.z = agregarConConjugado(S.z, nuevo);
    end
end

function S = borrarElem(S, tipo, idx)
% Borra el elemento y, si era complejo, tambien su conjugado.
    if strcmp(tipo,'polo'), v = S.p; else, v = S.z; end
    val = v(idx);
    v(idx) = [];
    if abs(imag(val)) > 1e-12
        [~, j] = min(abs(v - conj(val)));
        if ~isempty(j) && abs(v(j) - conj(val)) < 1e-6
            v(j) = [];
        end
    end
    if strcmp(tipo,'polo'), S.p = v; else, S.z = v; end
end

% ===================================================================
% DISENO DEL COMPENSADOR
% ===================================================================
function C = disenarCompensador(S, C)
% Calcula el compensador que hace pasar el lugar de las raices por la
% ubicacion sd elegida por el usuario.

    [num, den, ok, msg] = polinomios(S);
    if ~ok, fprintf('\n  >> %s\n', msg); return; end

    % --- 1. Obtener sd ---
    fprintf('\n--- Ubicacion deseada de los polos dominantes ---\n');
    fprintf('   1) Elegir con el raton (click en el plano s)\n');
    fprintf('   2) Escribir las coordenadas\n');
    fprintf('   3) Calcular desde especificaciones (zeta y wn)\n');
    fprintf('   0) Cancelar\n');
    modo = strtrim(leerTexto('Opcion: '));

    switch modo
        case '1'
            rootLocus(S, compVacio(), true);
            fprintf('\n  Haga click en la ubicacion deseada del polo dominante.\n');
            fprintf('  (Se tomara el semiplano superior; el conjugado es automatico.)\n');
            [x, y] = capturarClick();
            if isempty(x), fprintf('  >> Cancelado.\n'); return; end
            sd = x + 1i*abs(y);
        case '2'
            s = strtrim(leerTexto('  sigma, omega (parte real, parte imaginaria): '));
            v = str2num(['[' s ']']); %#ok<ST2NM>
            if numel(v) < 2, fprintf('  >> Entrada invalida.\n'); return; end
            sd = v(1) + 1i*abs(v(2));
        case '3'
            zeta = pedirEscalar('  Factor de amortiguamiento zeta (0<zeta<1): ', 0.5, false);
            wn   = pedirEscalar('  Frecuencia natural wn [rad/s]: ', 2, false);
            if zeta <= 0 || zeta >= 1
                fprintf('  >> zeta debe estar entre 0 y 1 para polos complejos.\n'); return;
            end
            % Polos dominantes de un segundo orden estandar:
            %   s = -zeta*wn +- j*wn*sqrt(1-zeta^2)
            sd = -zeta*wn + 1i*wn*sqrt(1-zeta^2);
            fprintf('  sd = %s   (Mp aprox %.1f %%, ts aprox %.3g s)\n', ...
                    num2strC(sd), 100*exp(-pi*zeta/sqrt(1-zeta^2)), 4/(zeta*wn));
        otherwise
            fprintf('  >> Cancelado.\n'); return;
    end

    if abs(imag(sd)) < 1e-9
        fprintf('  >> sd debe tener parte imaginaria (polo dominante complejo).\n');
        return;
    end

    % --- 2. Criterio del angulo: cuanto le falta a la planta ---
    [phi, angPlanta] = deficienciaAngular(S, sd);

    fprintf('\n===================================================================\n');
    fprintf('  CRITERIO DEL ANGULO EN sd = %s\n', num2strC(sd));
    fprintf('===================================================================\n');
    tablaAngulos(S, sd);
    fprintf('  ------------------------------------------------\n');
    fprintf('  Angulo de G(sd)                    = %+8.3f deg\n', angPlanta);
    fprintf('  Se requiere                        = -180.000 deg\n');
    fprintf('  DEFICIENCIA que aporta el compens. = %+8.3f deg\n', phi);
    fprintf('===================================================================\n');

    if abs(phi) < 1e-3
        fprintf('\n  sd YA esta sobre el lugar de las raices de la planta.\n');
        fprintf('  Basta con ajustar la ganancia: no hace falta cero ni polo.\n');
    elseif phi > 0
        fprintf('\n  Deficiencia POSITIVA: falta angulo -> compensador de ADELANTO\n');
        fprintf('  (o un PD). Un cero aporta angulo positivo y "jala" el lugar\n');
        fprintf('  de las raices hacia la izquierda, haciendo el sistema mas rapido.\n');
    else
        fprintf('\n  Deficiencia NEGATIVA: sobra angulo -> compensador de ATRASO\n');
        fprintf('  o reubicar sd. Un polo aporta angulo negativo.\n');
    end

    % --- 3. Tipo de compensador ---
    fprintf('\n--- Tipo de compensador ---\n');
    fprintf('   1) Ganancia pura (solo si sd ya esta sobre el lugar)\n');
    fprintf('   2) PD: Gc = Kc(s+zc)          [un cero]\n');
    fprintf('   3) Adelanto: Gc = Kc(s+zc)/(s+pc)   [metodo de la bisectriz]\n');
    fprintf('   4) Atraso: Gc = Kc(s+zc)/(s+pc)     [mejora el error estacionario]\n');
    fprintf('   0) Cancelar\n');
    tipo = strtrim(leerTexto('Opcion: '));

    switch tipo
        case '1'
            if abs(phi) > 1
                fprintf('  >> sd no esta sobre el lugar (deficiencia %.2f deg).\n', phi);
                fprintf('     Con ganancia pura no se puede llegar ahi.\n');
                return;
            end
            C.tipo = 'ganancia';  C.zc = [];  C.pc = [];

        case '2'
            % Un solo cero debe aportar todo el angulo phi.
            % angulo(sd + zc) = phi  ->  zc se despeja de la geometria.
            if phi <= 0 || phi >= 180
                fprintf('  >> Un cero solo puede aportar entre 0 y 180 grados.\n');
                return;
            end
            sg = real(sd); w = imag(sd);
            xz = sg - w/tan(phi*pi/180);
            C.tipo = 'pd';  C.zc = xz;  C.pc = [];
            fprintf('\n  Cero del PD en s = %.6g\n', xz);

        case '3'
            if phi <= 0
                fprintf('  >> El adelanto aporta angulo positivo; aqui la deficiencia\n');
                fprintf('     es negativa. Use atraso o mueva sd.\n');
                return;
            end
            [xz, xp] = bisectriz(sd, phi);
            C.tipo = 'adelanto';  C.zc = xz;  C.pc = xp;
            fprintf('\n  Metodo de la bisectriz:\n');
            fprintf('    cero en s = %.6g\n', xz);
            fprintf('    polo en s = %.6g\n', xp);
            fprintf('    Reparte el angulo requerido a ambos lados de la bisectriz\n');
            fprintf('    del angulo entre la horizontal y la recta sd-origen, que es\n');
            fprintf('    la configuracion que maximiza el adelanto de fase para una\n');
            fprintf('    separacion polo-cero dada.\n');

        case '4'
            % El atraso no busca cambiar la forma del lugar sino subir la
            % ganancia estatica. Se colocan cero y polo muy cerca del
            % origen, con una relacion igual a la mejora deseada, de modo
            % que el angulo que aportan sea despreciable en sd.
            beta = pedirEscalar('  Factor de mejora del error estacionario [10]: ', 10, false);
            if beta <= 1
                fprintf('  >> El factor debe ser mayor que 1.\n'); return;
            end
            xz = -abs(real(sd))/10;      % cero cerca del origen
            xp = xz/beta;                % polo aun mas cerca
            C.tipo = 'atraso';  C.zc = xz;  C.pc = xp;
            fprintf('\n  Cero en s = %.6g,  polo en s = %.6g  (relacion %g)\n', xz, xp, beta);
            fprintf('  Ambos muy cerca del origen: el angulo que aportan en sd es\n');
            fprintf('  casi nulo, asi que el lugar de las raices casi no cambia,\n');
            fprintf('  pero la ganancia estatica se multiplica por %g.\n', beta);

        otherwise
            fprintf('  >> Cancelado.\n'); return;
    end

    % --- 4. Criterio de la magnitud: fijar Kc ---
    % Se exige |Gc(sd)*G(sd)| = 1.
    C.Kc = 1;
    C.sd = sd;
    [numOL, denOL] = lazoAbierto(S, C);
    magOL = abs(polyval(numOL, sd) / polyval(denOL, sd));
    if magOL < 1e-300
        fprintf('  >> No se pudo calcular la ganancia (magnitud nula).\n');
        C = compVacio();  return;
    end
    C.Kc = 1/magOL;

    fprintf('\n  CRITERIO DE LA MAGNITUD:  |Gc(sd)*G(sd)| = 1\n');
    fprintf('    Kc = %.6g\n', C.Kc);
    fprintf('\n  COMPENSADOR RESULTANTE:  Gc(s) = %s\n', compStr(C));

    % --- 5. Verificacion ---
    [numOL, denOL] = lazoAbierto(S, C);
    c = ecCaracteristica(numOL, denOL, 1);
    r = roots(c);
    [err, j] = min(abs(r - sd));
    fprintf('\n  VERIFICACION: raiz mas cercana a sd = %s (error %.3g)\n', ...
            num2strC(r(j)), err);
    if err < 1e-6
        fprintf('  El polo de lazo cerrado quedo exactamente donde se pidio. [OK]\n');
    else
        fprintf('  ATENCION: no coincide exactamente, revise el diseno.\n');
    end

    rootLocus(S, C, true);
end

function [phi, angPlanta] = deficienciaAngular(S, sd)
% Angulo de G(sd) y cuanto le falta para llegar a -180 grados.
    angPlanta = 0;
    for k = 1:numel(S.z), angPlanta = angPlanta + angGrados(sd - S.z(k)); end
    for k = 1:numel(S.p), angPlanta = angPlanta - angGrados(sd - S.p(k)); end
    if S.Kg < 0, angPlanta = angPlanta + 180; end
    phi = -180 - angPlanta;
    phi = mod(phi + 180, 360) - 180;   % se lleva al rango (-180, 180]
end

function tablaAngulos(S, sd)
% Muestra el aporte de angulo y magnitud de cada polo y cero en sd.
% Es la forma mas clara de ver que el compensador "complementa" a la planta.
    fprintf('  Aporte de cada singularidad en sd:\n');
    fprintf('    %-8s %-14s %10s %12s\n', 'tipo', 'ubicacion', 'angulo', 'magnitud');
    for k = 1:numel(S.z)
        v = sd - S.z(k);
        fprintf('    %-8s %-14s %+9.3f  %11.4g\n', 'cero', num2strC(S.z(k)), ...
                angGrados(v), abs(v));
    end
    for k = 1:numel(S.p)
        v = sd - S.p(k);
        fprintf('    %-8s %-14s %+9.3f  %11.4g\n', 'polo', num2strC(S.p(k)), ...
                -angGrados(v), abs(v));
    end
    fprintf('    (los polos aportan angulo NEGATIVO, los ceros POSITIVO)\n');
end

function [xz, xp] = bisectriz(sd, phi)
% Metodo de la bisectriz para ubicar el cero y el polo de un compensador
% de adelanto que aporte exactamente phi grados en sd.
%
% Construccion: desde sd se trazan dos rayos, uno horizontal hacia la
% izquierda y otro hacia el origen. Se bisecta el angulo entre ellos y a
% partir de esa bisectriz se abren dos rayos separados +-phi/2, cuyas
% intersecciones con el eje real dan el cero y el polo.
    sg = real(sd);  w = imag(sd);
    th1 = angGrados(0 - sd);     % rayo de sd hacia el origen
    th2 = 180;                   % rayo horizontal hacia la izquierda
    thb = (th1 + th2)/2;         % bisectriz
    xz = corteEjeReal(sg, w, thb + phi/2);
    xp = corteEjeReal(sg, w, thb - phi/2);
end

function x = corteEjeReal(sg, w, thGrados)
% Interseccion con el eje real del rayo que sale de (sg, w) con angulo th.
% Parametrizando el rayo: punto = sd + t*exp(j*th); se impone parte
% imaginaria nula, w + t*sin(th) = 0, y se sustituye.
    t = tan(thGrados*pi/180);
    if abs(t) < 1e-12
        x = sign(cos(thGrados))*1e12;   % rayo horizontal: no corta
    else
        x = sg - w/t;
    end
end

function a = angGrados(v)
    a = atan2(imag(v), real(v))*180/pi;
end

% ===================================================================
% SISTEMA COMPENSADO
% ===================================================================
function sistemaCompensado(S, C)
    [num, den] = polinomios(S);
    [numOL, denOL] = lazoAbierto(S, C);
    c = ecCaracteristica(numOL, denOL, 1);

    fprintf('\n===================================================================\n');
    fprintf('  SISTEMA COMPENSADO\n');
    fprintf('===================================================================\n');

    fprintf('  PLANTA:\n');
    fprintf('    G(s)  = %g * %s / %s\n', S.Kg, factStr(S.z), factStr(S.p));
    fprintf('          = [%s] / [%s]\n\n', poli2str(num), poli2str(den));

    fprintf('  COMPENSADOR (tipo %s):\n', C.tipo);
    fprintf('    Gc(s) = %s\n', compStr(C));
    numC = C.Kc*poly(C.zc);  denC = poly(C.pc);
    fprintf('          = [%s] / [%s]\n\n', poli2str(real(numC)), poli2str(real(denC)));

    fprintf('  LAZO ABIERTO COMPENSADO  Gol(s) = Gc(s)*G(s):\n');
    fprintf('          = [%s] / [%s]\n\n', poli2str(numOL), poli2str(denOL));

    fprintf('  NUEVA ECUACION CARACTERISTICA  1 + Gc(s)G(s) = 0:\n');
    fprintf('    %s = 0\n\n', poli2str(c));

    r = roots(c);
    fprintf('  Polos de lazo cerrado del sistema compensado:\n');
    for k = 1:numel(r)
        marca = '';
        if ~isempty(C.sd) && (abs(r(k)-C.sd) < 1e-6 || abs(r(k)-conj(C.sd)) < 1e-6)
            marca = '   <- ubicacion deseada';
        end
        fprintf('    s%d = %-22s (Re = %+.6g)%s\n', k, num2strC(r(k)), real(r(k)), marca);
    end

    if all(real(r) < -1e-9)
        fprintf('\n  Todos los polos en el SPI: sistema compensado ESTABLE.\n');
    else
        fprintf('\n  ATENCION: hay polos fuera del SPI. Sistema NO estable.\n');
    end

    % Dominancia: los polos deseados solo gobiernan la respuesta si los
    % demas estan bastante mas a la izquierda.
    if ~isempty(C.sd)
        otros = r(abs(r-C.sd) > 1e-6 & abs(r-conj(C.sd)) > 1e-6);
        if ~isempty(otros)
            razon = min(abs(real(otros)))/abs(real(C.sd));
            fprintf('\n  Dominancia: el polo no deseado mas cercano esta %.2f veces\n', razon);
            fprintf('  mas a la izquierda que sd. ', razon);
            if razon > 5
                fprintf('Los polos deseados dominan claramente.\n');
            elseif razon > 3
                fprintf('Dominancia aceptable.\n');
            else
                fprintf('CUIDADO: no hay dominancia clara,\n');
                fprintf('  la respuesta real diferira de la del segundo orden ideal.\n');
            end
        end
    end

    % Error estacionario ante escalon: ess = 1/(1+Kp), con Kp = Gol(0).
    Kp = polyval(numOL, 0)/polyval(denOL, 0);
    if isfinite(Kp)
        fprintf('\n  Constante de posicion Kp = Gol(0) = %.6g\n', Kp);
        fprintf('  Error estacionario ante escalon: ess = 1/(1+Kp) = %.6g\n', 1/(1+Kp));
    else
        fprintf('\n  Hay integrador en el lazo: ess = 0 ante escalon.\n');
    end
    fprintf('===================================================================\n');
end

% ===================================================================
% RESPUESTA TEMPORAL
% ===================================================================
function compararRespuestas(S, C)
% Compara la respuesta al escalon del lazo cerrado sin y con compensador.
% La simulacion se hace integrando la ecuacion de estado con Runge-Kutta
% de cuarto orden, para no depender de step() ni de lsim().

    [num, den] = polinomios(S);

    % Lazo cerrado sin compensar, con la ganancia K actual
    numA = S.K*num;
    denA = ecCaracteristica(num, den, S.K);

    % Lazo cerrado compensado
    [numOL, denOL] = lazoAbierto(S, C);
    numB = numOL;
    denB = ecCaracteristica(numOL, denOL, 1);

    % Horizonte de simulacion: 8 constantes de tiempo del polo mas lento
    tFin = horizonte([roots(denA); roots(denB)]);
    N = 4000;

    [tA, yA, okA] = escalonRK4(numA, denA, tFin, N);
    [tB, yB, okB] = escalonRK4(numB, denB, tFin, N);

    figure('Name', 'Respuesta al escalon', 'NumberTitle', 'off');
    hold on; grid on;
    h = []; et = {};
    h(end+1) = plot([0 tFin], [1 1], '--', 'Color', [0.5 0.5 0.5]);
    et{end+1} = 'Referencia';
    if okA
        h(end+1) = plot(tA, yA, '-', 'LineWidth', 1.6, 'Color', [0.85 0.33 0.10]);
        et{end+1} = sprintf('Sin compensar (K=%g)', S.K);
    end
    if okB
        h(end+1) = plot(tB, yB, '-', 'LineWidth', 2, 'Color', [0 0.45 0.85]);
        et{end+1} = sprintf('Compensado (%s)', C.tipo);
    end
    xlabel('Tiempo [s]');  ylabel('Salida y(t)');
    title('Respuesta al escalon: planta sola vs sistema compensado');
    legend(h, et, 'Location', 'best');
    hold off; drawnow;

    fprintf('\n===================================================================\n');
    fprintf('  COMPARACION DE RESPUESTAS\n');
    fprintf('===================================================================\n');
    if okA, metricas('Sin compensar', tA, yA); end
    if okB, metricas('Compensado  ', tB, yB); end
    fprintf('===================================================================\n');
end

function tFin = horizonte(polos)
% Horizonte de simulacion a partir del polo estable mas lento.
    re = real(polos);
    est = re(re < -1e-6);
    if isempty(est)
        tFin = 10;   % sistema inestable: se muestra el arranque de la divergencia
    else
        tFin = min(120, max(2, 8/min(abs(est))));
    end
end

function [t, y, ok] = escalonRK4(numT, denT, tFin, N)
% Respuesta al escalon unitario de numT/denT, integrada con RK4 sobre la
% forma canonica controlable. Evita depender de step() o lsim().
    ok = false;  t = [];  y = [];
    denT = denT(:).';  numT = numT(:).';
    if numel(denT) < 2, return; end

    a0 = denT(1);
    denT = denT/a0;  numT = numT/a0;
    n = numel(denT) - 1;

    % Alineacion del numerador: b(1)*s^(n-1) + ... + b(n)
    b = [zeros(1, max(0, n - numel(numT))), numT];
    D = 0;
    if numel(b) > n           % funcion propia (no estrictamente): hay D
        D = b(1);
        b = b(2:end) - D*denT(2:end);
    end
    b = [zeros(1, n - numel(b)), b];

    % Forma canonica controlable
    A = zeros(n);
    A(1,:) = -denT(2:end);
    if n > 1
        A(2:n, 1:n-1) = eye(n-1);
    end
    Bv = zeros(n,1);  Bv(1) = 1;
    Cv = b;

    dt = tFin/N;
    t = (0:N)*dt;
    y = zeros(1, N+1);
    x = zeros(n,1);
    u = 1;                          % escalon unitario

    for i = 1:N+1
        y(i) = Cv*x + D*u;
        if ~isfinite(y(i)) || abs(y(i)) > 1e12
            y = y(1:i);  t = t(1:i);  ok = true;  return;   % divergio
        end
        k1 = A*x + Bv*u;
        k2 = A*(x + dt/2*k1) + Bv*u;
        k3 = A*(x + dt/2*k2) + Bv*u;
        k4 = A*(x + dt*k3)   + Bv*u;
        x = x + dt/6*(k1 + 2*k2 + 2*k3 + k4);
    end
    ok = true;
end

function metricas(etiqueta, t, y)
% Sobreimpulso, tiempo de asentamiento al 2 % y valor final.
    yf = y(end);
    if ~isfinite(yf) || abs(yf) > 1e10
        fprintf('  %s : la respuesta DIVERGE (sistema inestable)\n', etiqueta);
        return;
    end
    [ymax, imax] = max(y);
    if yf > 1e-9
        Mp = 100*(ymax - yf)/yf;
    else
        Mp = 0;
    end
    idx = find(abs(y - yf) > 0.02*abs(yf), 1, 'last');
    if isempty(idx), ts = 0; else, ts = t(min(idx+1, numel(t))); end
    fprintf('  %s : valor final = %.4g | Mp = %.2f %% | tp = %.4g s | ts(2%%) = %.4g s\n', ...
            etiqueta, yf, Mp, t(imax), ts);
end

% ===================================================================
% FORMATO
% ===================================================================
function s = compStr(C)
% Representacion legible del compensador.
    if strcmp(C.tipo, 'ninguno'), s = '1'; return; end
    if strcmp(C.tipo, 'ganancia')
        s = sprintf('%.6g', C.Kc); return;
    end
    sNum = sprintf('%.6g %s', C.Kc, factStr(C.zc));
    if isempty(C.pc)
        s = sNum;
    else
        s = sprintf('%s / %s', sNum, factStr(C.pc));
    end
end

function s = factStr(r)
% Forma factorizada: (s+2)(s+5) a partir de las raices.
    if isempty(r), s = '1'; return; end
    partes = cell(1, numel(r));
    for k = 1:numel(r)
        x = r(k);
        if abs(imag(x)) < 1e-12
            if real(x) < 0
                partes{k} = sprintf('(s+%.4g)', -real(x));
            elseif real(x) > 0
                partes{k} = sprintf('(s-%.4g)', real(x));
            else
                partes{k} = 's';
            end
        else
            partes{k} = sprintf('(s-(%s))', num2strC(x));
        end
    end
    s = strjoin_compat(partes, '');
end

function s = num2strC(x)
    if abs(imag(x)) < 1e-12
        s = sprintf('%.6g', real(x));
    elseif imag(x) > 0
        s = sprintf('%.6g+%.6gi', real(x), imag(x));
    else
        s = sprintf('%.6g-%.6gi', real(x), -imag(x));
    end
end

function s = vec2str(v)
    if isempty(v), s = '(ninguno)'; return; end
    partes = cell(1, numel(v));
    for k = 1:numel(v), partes{k} = num2strC(v(k)); end
    s = ['[' strjoin_compat(partes, ', ') ']'];
end

function s = poli2str(c)
    n = numel(c) - 1;  s = '';
    for k = 1:numel(c)
        a = c(k);  e = n - k + 1;
        if abs(a) < 1e-12, continue; end
        if isempty(s)
            signo = ''; if a < 0, signo = '-'; end
        else
            if a < 0, signo = ' - '; else, signo = ' + '; end
        end
        mag = abs(a);
        if e == 0
            term = sprintf('%.6g', mag);
        elseif e == 1
            if abs(mag-1) < 1e-12, term = 's'; else, term = sprintf('%.6g s', mag); end
        else
            if abs(mag-1) < 1e-12, term = sprintf('s^%d', e);
            else, term = sprintf('%.6g s^%d', mag, e); end
        end
        s = [s signo term]; %#ok<AGROW>
    end
    if isempty(s), s = '0'; end
end

function s = strjoin_compat(partes, sep)
    s = partes{1};
    for k = 2:numel(partes), s = [s sep partes{k}]; end %#ok<AGROW>
end
