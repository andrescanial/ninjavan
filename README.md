-- https://redash.ninjavan.co/queries/19001/source

SELECT 
    s.id AS shipment_id
    ,s.orig_hub_id
    , oh.name AS orig_hub_name
    , s.dest_hub_id
    , dh.name AS dest_hub_name
    , s.curr_hub_id
    , ch.name AS curr_hub_name
    , s.status, s.comments
    , DATE(s.created_at + interval 8 hour) AS shipment_create_date
    , s.created_at + interval 8 hour AS shipment_create_datetime
    , ev.created_at + interval 8 hour AS shipment_hub_inbound_datetime
    , DATE(ev.created_at + interval 8 hour) AS shipment_hub_inbound_date
    , COUNT(*) AS add_to_shipment_orders
    , ev.event
    , case when s.dest_hub_id in (92,96,100,115,122,155,239,415,439,478,668,704,758,1029,1058,104224,104249) then 'CONSOL-Tacloban'
        when s.dest_hub_id in (36,179,191,193,263,407,431,451,482,602,632,711,723,893,894,927,1002,1149,104328) then 'CONSOL-Iloilo'
        when s.dest_hub_id in (32,143,201,365,369,486,640,680,1024,1080) then 'CONSOL-Bacolod'
        when s.dest_hub_id in (30,135,137,141,161,175,197,203,217,494,534,648,664,700,755,763,869,928,949,982,1013,1037,104229,104162) then 'CONSOL-CGY'
        when s.dest_hub_id in (139,157,169,243,502,749,996,104230) then 'CONSOL-Zamboanga'
        when s.dest_hub_id in (205,506,675,759,989) then 'CONSOL-Tagbilaran'
        when s.dest_hub_id in (124,411,510,636,691,104204) then 'CONSOL-Dumaguete'
        when s.dest_hub_id in (353,620,686,735,806,845) then 'CONSOL-TGG'
        when s.dest_hub_id in (185,628,825,826,827,104237,104236,104238,104235) then 'CONSOL-Puerto Princesa'
        when s.dest_hub_id in (7,131,159,251,306,423,824,840,842,843,862,994,1008,103206,103207) then 'CONSOL-Batangas'
        when s.dest_hub_id in (88,106,108,181,333,435,466,753,812,817,933,966,997,998,1009,1020,1056,104234,104233) then 'WH-BAYOMBONG'
        when s.dest_hub_id in (84,102,109,227,455,459,474,712,745,864,865,991,1022,1123,1124,1167) then 'WH-DAET'
        when s.dest_hub_id in (133,209,213,992,993,1011,103394,104241,104239,104243,104240) then 'CONSOL-Sorsogon'
        when s.dest_hub_id in (1,11,72,80,117,126,149,215,219,229,235,237,241,244,255,259,282,285,337,403,419,427,530,542,546,550,554,558,562,594,656,692,696,705,707,746,747,748,750,767,790,800,803,821,823,833,844,847,848,859,860,861,870,873,888,910,911,937,979,999,1000,1014,1015,1017,1019,1026,1034,1035,1036,1039,1047,1064,1072,1093,1162,103261,103277,103369,104244,104248,104242,104245,104246,104247) then 'WH-CBY2'
        when s.dest_hub_id in (26,163,165,171,173,177,195,221,387,498,690,702,986,990,103188,104227,104330,104228) then 'WH-DAVAO'
        when s.dest_hub_id in (48,52,56,64,68,76,104,183,187,189,207,225,325,379,717,718,719,725,834,983,984,1003,1005,1006,1018,1025,1027,1028,1030,1031,1044,1050,103556,103605,104231) then 'WH-ANGELES'
        when s.dest_hub_id in (3,22,44,111,199,245,274,278,291,297,303,313,345,566,570,574,578,582,586,590,598,604,612,616,652,672,683,685,689,693,694,697,701,706,720,724,730,734,736,737,738,741,742,789,802,816,819,846,851,852,856,866,868,872,889,903,921,962,1001,1007,1010,1038,1046,1049,1051,1057,1077,1109,1113,1139,1173,103399,103423,104095,104161,104232,104338) then 'WH-NORTH METRO'
        when s.dest_hub_id in (15,120,270,288,490,687,688,751,871,985,1023,1054,103187,104226,104225,104329) then 'WH-CEBU'
        end as consol_hub
    , case when COUNT(*) >=2 then 'sack'
        when COUNT(*) = 1 and s.comments is null then 'bulky'
        when COUNT(*) = 1 and s.comments REGEXP  'seal|S01|mailer|b2b|PPOD|L001' then 'sack'
        else 'bulky'
        end as shipment_category


FROM hub_prod_gl.shipments s
join hub_prod_gl.shipment_orders so on so.shipment_id = s.id 
LEFT JOIN sort_prod_gl.hubs oh ON oh.hub_id=s.orig_hub_id and oh.system_id='ph'
LEFT JOIN sort_prod_gl.hubs dh ON dh.hub_id=s.dest_hub_id and dh.system_id='ph'
LEFT JOIN sort_prod_gl.hubs ch ON ch.hub_id=s.curr_hub_id and ch.system_id='ph'


left join hub_prod_gl.shipment_events ev on ev.shipment_id = s.id 
    and ev.event IN ('SHIPMENT_VAN_INBOUND','SHIPMENT_HUB_INBOUND', 'SHIPMENT_CLOSED', 'SHIPMENT_CREATED', 'SHIPMENT_REOPENED')
    and ev.hub_system_id = 'ph'
left join hub_prod_gl.shipment_events ev1 on ev1.shipment_id = ev.shipment_id 
    and ev1.event IN ('SHIPMENT_VAN_INBOUND','SHIPMENT_HUB_INBOUND', 'SHIPMENT_CLOSED', 'SHIPMENT_CREATED', 'SHIPMENT_REOPENED') 
    and ev1.hub_system_id = 'ph'
    and ev.id < ev1.id


WHERE s.curr_hub_id in (
                        select hub_id 
                        FROM sort_prod_gl.hubs
                        where system_id='ph'
                            and sort_hub = 1
                        )
    and lower(s.curr_hub_country) = 'ph'
    and s.created_at >= now() - interval {{start}} day -- change from 10 to 15 days (camille) -- changed to parameter (dust)
    and s.created_at < now() - interval {{hour}} hour
    and s.deleted_at is null
    AND so.deleted_at is null
    and s.status IN ('Closed','At Transit Hub', 'Pending')
    AND ev.event IN ('SHIPMENT_HUB_INBOUND', 'SHIPMENT_CLOSED', 'SHIPMENT_CREATED', 'SHIPMENT_REOPENED')
    AND ev.event <> 'SHIPMENT_STATUS_REVERTED'  -- edit (daphne BI)
    AND ev1.id is NULL
    
GROUP BY 1
