# I'll also write english comment later - 
def first_step(a):       # legelső lépés (Ezt beépítem majd a fŐ_függvénybe)
    a = (a
    .assign(gól_arány=lambda df_:df_.HG-df_.AG)
     )
    
    a = (a
    .assign(home_win = [1 if  x>0 else 0 for x in a.gól_arány],
           draw = [1 if x == 0 else 0 for x in a.gól_arány],
           away_win = [1 if  x<0 else 0 for x in a.gól_arány])
    )
    
    return a
   
def clean_gólok(a):        #ebben összahozom a lőtt és kapott gólokat (nem könnyű, mert az otthon lőtt gólokhoz az  idegenben lőtteket kell hozzáadni/csapat)
    gyűjtő = {}
    gólok = []
   
    teams = [x for x in a.Ht.unique()]
    teams2 = [x for x in a.At.unique() if x not in teams]
    teams = teams+teams2
    
    for t in teams:
        dd = a[a.Ht == t] 
        gólok.append(dd.HG.values.sum())
        bb = a[a.At == t]
        gólok.append(bb.AG.values.sum())
        gólok = [sum(gólok)]
        gyűjtő[t] = gólok
        
        gólok = []
    
    _goal = pd.DataFrame(gyűjtő).T
    
    gyűjtő2 = {}
    gólok2 = []
   
    for t in teams:
        dd = a[a.Ht == t] 
        gólok2.append(dd.AG.values.sum())
        bb = a[a.At == t]
        gólok2.append(bb.HG.values.sum())
        gólok2 = [sum(gólok2)]
        gyűjtő2[t] = gólok2
        
        gólok2 = []
        
    _conc_goal = pd.DataFrame(gyűjtő2).T
    
    tG = [x[0] for x in gyűjtő.values()]
    tC = [x[0] for x in gyűjtő2.values()]
    
    vége = {'G':tG,'CG':tC}
    
    gt = pd.DataFrame(vége, index=teams).reset_index().rename(columns={'index':'team'})
    gt ['p_m'] = gt['G']-gt['CG']
    
    return gt
 
def make_table(a):     #tabella_készítő (a=dataframe)
    
    a1 = clean_gólok(a).sort_values(by='team')
    
    hdata = a.groupby('Ht')[['gól_arány','home_win','draw','away_win']].sum()
    adata = a.groupby('At')[['gól_arány','home_win','draw','away_win']].sum().rename(columns={'home_win':'away_win','away_win':'home_win'})
    
    b = pd.concat([hdata,adata]).reset_index().rename(columns={'index':'team'}).groupby('team').sum().reset_index()
    b = b.sort_values(by='team')
    
    a1b = pd.concat([a1,b]).groupby('team').sum().reset_index()
    
    a1b = (a1b
        .assign(pont = lambda x: x.home_win*3 + x.draw*1,
               MP= lambda x: x.home_win + x.draw + x.away_win)
        #.select_dtypes('float64').astype('uint8')
          )
    
    
    a2b = a1b.sort_values(['pont','home_win','p_m','G'], ascending=False).reset_index()
    col = ['team','MP','home_win','draw','away_win','G','CG','p_m','pont']
    a2b= (a2b[col[1:]]
              
             .astype('int')
              .assign(team=a2b.team)
         )
    
    a2b.index = np.arange(1,len(a2b)+1,1)
    
    return a2b[col]
    
    
def round_position(a,b,f):       # itt megkaphatjuk a fordulónkénti helyezéseket (a=df, b=kért csapat, f=hány meccs van egy fordulóban (csapat/2))
    accumulator = []
    
    for i in range(f-1,len(a),f):
            f = make_table(a.loc[:i,:])
            accumulator.append(f[f.team==b].index[0])
            
    return accumulator

def pos_last_round(a,b):        #az aktuális helyezése a csapatnak (a=df, b=kért csapat) - ez majd a DixonCole modellhez kell
    f = make_table(a)
    
    return f[f.team == b].index[0]
    
 
