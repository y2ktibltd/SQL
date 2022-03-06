SELECT
    *
FROM
    (
        SELECT
            q1."Dock",
            q1."Time In",
            q1.trip_name,
            decode(q2.name, 'Pallets', q1.not_loaded)           not_loaded,
            decode(q2.name, 'Pallets', q1.loaded)               loaded
        FROM
            (
                SELECT
                    COUNT(
                        CASE
                            WHEN wlpn1.lpn_context = 11 THEN
                                wlpn1.license_plate_number
                        END
                    )                                                    AS not_loaded,
                    COUNT(
                        CASE
                            WHEN wlpn1.lpn_context = 9 THEN
                                wlpn1.license_plate_number
                        END
                    )                                                    AS loaded,
                    xwtd1.trip_name,
                    df.dock_number                                       AS "Dock",
                    to_char(df.date_time_in, 'MM/DD/YY hh:mi:ss')        AS "Time In"
                FROM
                    obianew_sds_ebs_12_1_3.xxbhp_wms_trips_dim_v        xwtd1,
                    obianew_sds_ebs_12_1_3.wsh_delivery_assignments     wda1,
                    obianew_sds_ebs_12_1_3.wsh_delivery_details         wdd1,
                    obianew_sds_ebs_12_1_3.wms_license_plate_numbers    wlpn1,
                    obianew_sds_ebs_12_1_3.dock_file                    df
                WHERE
                        wda1.delivery_id = xwtd1.delivery_id
                    AND wda1.delivery_detail_id = wdd1.delivery_detail_id
                    AND wdd1.lpn_id = wlpn1.lpn_id
                    AND xwtd1.trip_name = df.shipment_id
                    AND 1 = 1
                    AND xwtd1.delivery_id IS NOT NULL
                    AND df.organization_code = 'GPD'
                GROUP BY
                    xwtd1.trip_name,
                    df.dock_number,
                    to_char(df.date_time_in, 'MM/DD/YY hh:mi:ss')
            )  q1,
            (
                SELECT
                    'Pallets' AS name
                FROM
                    dual
            )  q2
    )
ORDER BY
    "Dock"
