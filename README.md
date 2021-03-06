%========================================================================
%----This outputs TPTP Problem Set clauses and formulae in a format
%----acceptable to the leanCoP system.
%----
%----Written by Jens Otten, November 2000
%----extended to handle FOF: JO, June 2004
%========================================================================

%%% Thomas Raths (TR) November, 2005-: changes due to TPTP3.1.0-
:-set_flag(print_depth,1000). %%% for Eclipse Prolog

%----------------------------------------------------------------------
%----Print out a literal with - for negative, nothing for positive.
%----Use positive representation
%%% T.R.
output_leancop_signed_literal(--('$tptp_equal'(X,Y))) :-
    !, write(' (equal('), write(X), write(' , '), write(Y), write('))').

output_leancop_signed_literal(--('$tptp_not_equal'(X,Y))) :-
    !, write(' - (equal('), write(X), write(' , '), write(Y), write('))').

output_leancop_signed_literal(++('$tptp_equal'(X,Y))) :-
    !, write(' - (equal('), write(X), write(' , '), write(Y), write('))').

output_leancop_signed_literal(++('$tptp_not_equal'(X,Y))) :-
    !, write(' (equal('), write(X), write(' , '), write(Y), write('))').

output_leancop_signed_literal(--Atom):-
    !, write(' '), write(Atom).

output_leancop_signed_literal(++Atom):-
    write('-'), write(Atom).
%----------------------------------------------------------------------
%----Print out the literals of a clause in leanCoP format.
%----Special case of an empty clause
output_leancop_literals([]):-
    write('[]').

output_leancop_literals([OneLiteral]):-
    !, output_leancop_signed_literal(OneLiteral).

output_leancop_literals([FirstLiteral|RestOfLiterals]):-
    output_leancop_signed_literal(FirstLiteral),
    write('  ,'), nl, write(' '),
    output_leancop_literals(RestOfLiterals).
%----------------------------------------------------------------------
%----Print out the clauses in leanCoP format.
output_leancop_clauses([]).
%%% for TPTP-v3.1.0 or later
%%% if clause = $true (or ­($false)), then ignore it,
%%% if clause = $false (or -($true)), then clause set is valid => add e.g. [truexxx], [-(truexxx)]
%%% for TPTP-v3.2.0: '$true', '$false', for TPTP-v3.1.0: $true, $false

output_leancop_clauses([cnf(Name,Status,[Literal])|
RestOfClauses]):- 
    (Literal = ++('$true'); 
     Literal = ++($true);
     Literal = --('$false');
     Literal = --($false)), !,
    output_leancop_clauses(RestOfClauses).

output_leancop_clauses([cnf(Name,Status,[Literal])|
RestOfClauses]) :-
    (Literal = ++('$false'); 
     Literal = ++($false);
     Literal = --('$true');
     Literal = --($true)), !,
    write('% '), write(Name), write(', '),
    write(Status), write('.'), nl,
    write('[truexxx], [-(truexxx)]'),
    (RestOfClauses\==[]  ->
        (nl, nl, write('  ,'), nl, nl);
         true),
    output_leancop_clauses(RestOfClauses).

output_leancop_clauses([cnf(Name,Status,Literals)|
RestOfClauses]):-
    write('% '), write(Name), write(', '),
    write(Status), write('.'), nl,
    write('['),
    output_leancop_literals(Literals),
    write(']'),
    (RestOfClauses\==[]  ->
        (nl, nl, write('  ,'), nl, nl);
         true),
    output_leancop_clauses(RestOfClauses).

%%% for TPTP-v2.7.0 or earlier
output_leancop_clauses([input_clause(Name,Status,Literals)|
RestOfClauses]):-
        output_leancop_clauses([cnf(Name,Status,Literals)|
    RestOfClauses]).
%----------------------------------------------------------------------
%----Print out the list of input clauses as a formula in leanCoP format.
output_leancop_formula([]):-
    !.

output_leancop_formula(Clauses):-
    nl,
    write('f(['), nl, nl,
    output_leancop_clauses(Clauses), nl, nl,
    write(']).'), nl, nl.
%----------------------------------------------------------------------

%----------------------------------------------------------------------
%----Print out the connectives, quantifiers, and literals of a formula
%----in leanCoP format.
output_leancop_fof((~ A)):-
    !, write('( ~ '), output_leancop_fof(A), write(' )').
output_leancop_fof('|'(A,B) ):-
    !, write('( '), output_leancop_fof(A), write(' ; '),
    output_leancop_fof(B), write(' )').
output_leancop_fof((A;B)):-
    !, write('( '), output_leancop_fof(A), write(' ; '),
    output_leancop_fof(B), write(' )').
output_leancop_fof((A & B)):-
    !, write('( '), output_leancop_fof(A), write(' , '),
    output_leancop_fof(B), write(' )').
output_leancop_fof((A => B)):-
    !, write('( '), output_leancop_fof(A), write(' => '),
    output_leancop_fof(B), write(' )').
output_leancop_fof((A <=> B)):-
    !, write('( '), output_leancop_fof(A), write(' <=> '),
    output_leancop_fof(B), write(' )').
output_leancop_fof((! [] : A)):- !, output_leancop_fof(A).
output_leancop_fof((! [V|L] : A)):-
    !, write('( all '), print(V), write(' : '),
    output_leancop_fof(! L : A), write(' )').
output_leancop_fof((? [] : A)):- !, output_leancop_fof(A).
output_leancop_fof((? [V|L] : A)):-
    !, write('( ex '), print(V), write(' : '),
    output_leancop_fof(? L : A), write(' )').
output_leancop_fof('$true') :- !, write('(true___=>true___)').
output_leancop_fof($true) :- !, write('(true___=>true___)'). % TPTP 3.1.0
output_leancop_fof('$false') :- !, write('(false___ , ~ false___)').
output_leancop_fof($false) :- !, write('(false___ , ~ false___)'). % TPTP 3.1.0
output_leancop_fof('$tptp_equal'(X,Y)) :- !, write('equal('), 
                                           write(X), write(' , '), write(Y), write(')').
output_leancop_fof('$tptp_not_equal'(X,Y)) :- !,  write('(~ ( equal('), 
                                           write(X), write(' , '), write(Y), write(')))').
output_leancop_fof(Atom) :- print(Atom).
%----------------------------------------------------------------------
%----Print out the formulae in leanCoP format.
output_leancop_fo_formulae([]).

%%% for TPTP-v3.1.0 or later
output_leancop_fo_formulae([fof(Name,Status,Formula)|RestOfFormulae]) :-
    (Status==conjecture, RestOfFormulae \= []) -> 
      (append(RestOfFormulae,[fof(Name,Status,Formula)],Formulae),
       output_leancop_fo_formulae(Formulae)) ;
      (write('% '), write(Name), write(', '), write(Status), write('.'), nl,
       write('('), output_leancop_fof(Formula), write(')'),
       (RestOfFormulae == [] -> true;
        (((RestOfFormulae=[fof(_,conjecture,_)]  ->
           (nl, nl, write('  =>'), nl, nl)); 
           (nl, nl, write('  ,'), nl, nl)),
          output_leancop_fo_formulae(RestOfFormulae)))).




%%% for TPTP-v2.7.0 or earlier
output_leancop_fo_formulae([input_formula(Name,Status,Formula)|
RestOfFormulae]):-
    output_leancop_fo_formulae([fof(Name,Status,Formula)|RestOfFormulae]).

%----------------------------------------------------------------------
%----Print out the list of input formulae as a first-order formula in
%----leanCoP format.
output_leancop_fo_formula([]):-
    !.

output_leancop_fo_formula(Formulae):-
    nl,
    write('f(('), nl, nl,
    %%% negate problems without conjecture
    (\+ (member(fof(_,conjecture,_),Formulae);
         member(input_formula(_,conjecture,_),Formulae)) -> 
                                             (write('~ ('), nl) ; true),
    output_leancop_fo_formulae(Formulae), nl, nl,
    (\+ (member(fof(_,conjecture,_),Formulae);
         member(input_formula(_,conjecture,_),Formulae)) -> 
                                             (write(')'), nl); true),
    write(')).'), nl, nl.

%----------------------------------------------------------------------

%----------------------------------------------------------------------
%----Print out all the clauses in leanCoP format.
leancop(leancop,Clauses,_):-
    tptp_clauses(Clauses),
    output_leancop_formula(Clauses).

%----Print out first-order formula in leanCoP format.
leancop(leancop,Formulae,_):-
    tptp_formulae(Formulae),
    output_leancop_fo_formula(Formulae).
%----------------------------------------------------------------------
%----Provide information about the leanCoP format.
leancop_format_information('%','.leancop').
%----------------------------------------------------------------------
%----Provide information about the TPTP file.
leancop_file_information(format,leancop).
%----------------------------------------------------------------------w
